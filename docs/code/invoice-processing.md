---
title: "Invoice Processing — Source Code"
description: "Azure infrastructure setup script and Python invoice processor for the Invoice Processor project."
date: 2026-04-23
tags:
  - python
  - azure
  - automation
status: published
---

# Invoice Processing — Source Code

Source files for the [Invoice Processor](../projects/invoice-processor.md) project. Two files: the Azure provisioning script and the main Python processor.

---

## setup.sh

Provisions all required Azure resources — resource group, Document Intelligence account, storage account, and both blob containers — then writes the `.env` credentials file.

```bash title="setup.sh" linenums="1"
#!/usr/bin/env bash
set -euo pipefail

RG="rg-fn-invoice-processing"
DI_NAME="fn-doc-intelligence"
STORAGE="fninvoicestorage"
LOCATION="eastus"
INPUT_CONTAINER="input"
OUTPUT_CONTAINER="output"

echo "==> Creating resource group: $RG"
az group create --name "$RG" --location "$LOCATION"

echo "==> Creating Document Intelligence resource: $DI_NAME"
az cognitiveservices account create \
  --name "$DI_NAME" \
  --resource-group "$RG" \
  --kind FormRecognizer \
  --sku S0 \
  --location "$LOCATION" \
  --yes

echo "==> Retrieving Document Intelligence endpoint and key"
DI_ENDPOINT=$(az cognitiveservices account show \
  --name "$DI_NAME" \
  --resource-group "$RG" \
  --query "properties.endpoint" \
  --output tsv)

DI_KEY=$(az cognitiveservices account keys list \
  --name "$DI_NAME" \
  --resource-group "$RG" \
  --query "key1" \
  --output tsv)

echo "==> Creating Storage Account: $STORAGE"
az storage account create \
  --name "$STORAGE" \
  --resource-group "$RG" \
  --location "$LOCATION" \
  --sku Standard_LRS \
  --kind StorageV2 \
  --allow-blob-public-access false

echo "==> Retrieving storage connection string"
STORAGE_CONN_STR=$(az storage account show-connection-string \
  --name "$STORAGE" \
  --resource-group "$RG" \
  --query "connectionString" \
  --output tsv)

echo "==> Creating blob containers: $INPUT_CONTAINER, $OUTPUT_CONTAINER"
az storage container create --name "$INPUT_CONTAINER" --connection-string "$STORAGE_CONN_STR"
az storage container create --name "$OUTPUT_CONTAINER" --connection-string "$STORAGE_CONN_STR"

ENV_FILE="$(dirname "$0")/.env"

if [[ -f "$ENV_FILE" ]]; then
  echo ""
  echo "ERROR: $ENV_FILE already exists. Remove it manually before re-running setup."
  exit 1
fi

cat > "$ENV_FILE" <<EOF
AZURE_DI_ENDPOINT=$DI_ENDPOINT
AZURE_DI_KEY=$DI_KEY
AZURE_STORAGE_CONNECTION_STRING=$STORAGE_CONN_STR
AZURE_STORAGE_INPUT_CONTAINER=$INPUT_CONTAINER
AZURE_STORAGE_OUTPUT_CONTAINER=$OUTPUT_CONTAINER
EOF

chmod 600 "$ENV_FILE"
echo ""
echo "==> .env written to $ENV_FILE (permissions: 600)"
```

---

## invoice_processor.py

Main script. Reads invoice files from the input container, runs each through Azure Document Intelligence, and writes a consolidated CSV to the output container.

```python title="invoice_processor.py" linenums="1"
import csv
import io
import os
from datetime import datetime, UTC
from pathlib import Path

from azure.ai.documentintelligence import DocumentIntelligenceClient
from azure.core.credentials import AzureKeyCredential
from azure.storage.blob import BlobServiceClient
from dotenv import load_dotenv

load_dotenv(Path(__file__).resolve().parent / ".env")

DI_ENDPOINT = os.environ["AZURE_DI_ENDPOINT"]
DI_KEY = os.environ["AZURE_DI_KEY"]
STORAGE_CONN_STR = os.environ["AZURE_STORAGE_CONNECTION_STRING"]
INPUT_CONTAINER = os.getenv("AZURE_STORAGE_INPUT_CONTAINER", "input")
OUTPUT_CONTAINER = os.getenv("AZURE_STORAGE_OUTPUT_CONTAINER", "output")

SUPPORTED_EXTENSIONS = (".pdf", ".jpg", ".jpeg", ".png", ".tiff", ".bmp")

CSV_FIELDS = [
    "source",
    "vendor",
    "invoice_date",
    "line_description",
    "quantity",
    "unit_price",
    "line_amount",
    "invoice_total",
]


def analyze_invoice(di_client, pdf_bytes):
    poller = di_client.begin_analyze_document(
        "prebuilt-invoice",
        body=pdf_bytes,
        content_type="application/octet-stream",
    )
    return poller.result()


def extract_rows(result, source_name):
    rows = []
    for doc in result.documents:
        fields = doc.fields or {}

        vendor_field = fields.get("VendorName")
        vendor = vendor_field.value_string if vendor_field else ""

        date_field = fields.get("InvoiceDate")
        date = str(date_field.value_date) if (date_field and date_field.value_date) else ""

        total_field = fields.get("InvoiceTotal")
        total = ""
        if total_field and total_field.value_currency:
            total = str(total_field.value_currency.amount)

        items_field = fields.get("Items")
        items = items_field.value_array if (items_field and items_field.value_array) else []

        if items:
            for item in items:
                item_fields = item.value_object or {}

                desc_field = item_fields.get("Description")
                desc = desc_field.value_string if desc_field else ""

                qty_field = item_fields.get("Quantity")
                qty = str(qty_field.value_number) if (qty_field and qty_field.value_number is not None) else ""

                up_field = item_fields.get("UnitPrice")
                unit_price = str(up_field.value_currency.amount) if (up_field and up_field.value_currency) else ""

                amt_field = item_fields.get("Amount")
                amount = str(amt_field.value_currency.amount) if (amt_field and amt_field.value_currency) else ""

                rows.append({
                    "source": source_name,
                    "vendor": vendor,
                    "invoice_date": date,
                    "line_description": desc,
                    "quantity": qty,
                    "unit_price": unit_price,
                    "line_amount": amount,
                    "invoice_total": total,
                })
        else:
            rows.append({
                "source": source_name,
                "vendor": vendor,
                "invoice_date": date,
                "line_description": "",
                "quantity": "",
                "unit_price": "",
                "line_amount": "",
                "invoice_total": total,
            })

    return rows


def main():
    di_client = DocumentIntelligenceClient(DI_ENDPOINT, AzureKeyCredential(DI_KEY))
    blob_service = BlobServiceClient.from_connection_string(STORAGE_CONN_STR)
    input_client = blob_service.get_container_client(INPUT_CONTAINER)
    output_client = blob_service.get_container_client(OUTPUT_CONTAINER)

    blobs = [b for b in input_client.list_blobs() if b.name.lower().endswith(SUPPORTED_EXTENSIONS)]

    if not blobs:
        print(f"No invoice files found in container '{INPUT_CONTAINER}'.")
        return

    all_rows = []
    failed = []
    for blob in blobs:
        print(f"Processing: {blob.name}")
        try:
            data = input_client.download_blob(blob.name).readall()
            result = analyze_invoice(di_client, data)
            rows = extract_rows(result, blob.name)
            all_rows.extend(rows)
            print(f"  -> {len(rows)} row(s) extracted")
        except Exception as e:
            print(f"  ERROR: {e}")
            failed.append(blob.name)

    if failed:
        print(f"\nFailed ({len(failed)}): {', '.join(failed)}")

    if not all_rows:
        print("No data extracted.")
        return

    csv_buffer = io.StringIO()
    writer = csv.DictWriter(csv_buffer, fieldnames=CSV_FIELDS)
    writer.writeheader()
    writer.writerows(all_rows)

    timestamp = datetime.now(UTC).strftime("%Y%m%d-%H%M%S")
    output_blob_name = f"invoices-{timestamp}.csv"
    output_client.upload_blob(output_blob_name, csv_buffer.getvalue(), overwrite=True)
    print(f"\nResults written to '{OUTPUT_CONTAINER}/{output_blob_name}'")


if __name__ == "__main__":
    main()
```
