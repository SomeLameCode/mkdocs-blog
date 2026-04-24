---
title: "Transcript Action Extractor — Source Code"
description: "Azure infrastructure setup script and Python action item extractor for the Transcript Action Extractor article."
date: 2026-04-24
tags:
  - python
  - azure
  - azure-openai
  - automation
  - workplace
status: published
---

# Transcript Action Extractor — Source Code

Source files for the [Transcript Action Extractor](../articles/code/transcript-action-extractor.md) article. Two files: the Azure provisioning script and the main Python extractor.

---

## setup.sh

Provisions the full Azure OpenAI stack — resource group, Azure OpenAI resource, and gpt-4o model deployment — then reads credentials back from Azure and writes them to `.env`. No manual portal steps required. Safe to re-run (idempotent).

```bash title="setup.sh" linenums="1"
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
ENV_FILE="$SCRIPT_DIR/.env"

# --- Configuration -------------------------------------------------------
RESOURCE_GROUP="rg-fn-openai-transcript"
LOCATION="australiaeast"
RESOURCE_NAME="aoai-fn-transcript-1234"
DEPLOYMENT_NAME="transcript"
MODEL_NAME="gpt-4o"
MODEL_VERSION="2024-11-20"
API_VERSION="2024-06-01"
SKU_CAPACITY=10          # thousands of tokens per minute
# -------------------------------------------------------------------------

# Verify az CLI is available and the user is logged in
if ! command -v az &>/dev/null; then
    echo "Error: Azure CLI not found." >&2
    echo "Install it from https://docs.microsoft.com/cli/azure/install-azure-cli" >&2
    exit 1
fi

if ! az account show &>/dev/null 2>&1; then
    echo "Error: not logged in to Azure. Run: az login" >&2
    exit 1
fi

echo "Subscription: $(az account show --query '[name, id]' -o tsv | tr '\t' ' / ')"
echo ""

# Resource group (az group create is idempotent)
echo "==> Resource group: $RESOURCE_GROUP"
az group create --name "$RESOURCE_GROUP" --location "$LOCATION" --output none
echo "    OK"

# Azure OpenAI resource
echo "==> Azure OpenAI resource: $RESOURCE_NAME"
if az cognitiveservices account show \
        --name "$RESOURCE_NAME" \
        --resource-group "$RESOURCE_GROUP" \
        &>/dev/null 2>&1; then
    echo "    Already exists, skipping creation"
else
    az cognitiveservices account create \
        --name "$RESOURCE_NAME" \
        --resource-group "$RESOURCE_GROUP" \
        --kind "OpenAI" \
        --sku "S0" \
        --location "$LOCATION" \
        --yes \
        --output none
    echo "    Created"
fi

# Model deployment
echo "==> Model deployment: $DEPLOYMENT_NAME ($MODEL_NAME $MODEL_VERSION)"
if az cognitiveservices account deployment show \
        --name "$RESOURCE_NAME" \
        --resource-group "$RESOURCE_GROUP" \
        --deployment-name "$DEPLOYMENT_NAME" \
        &>/dev/null 2>&1; then
    echo "    Already exists, skipping deployment"
else
    az cognitiveservices account deployment create \
        --resource-group "$RESOURCE_GROUP" \
        --name "$RESOURCE_NAME" \
        --deployment-name "$DEPLOYMENT_NAME" \
        --model-name "$MODEL_NAME" \
        --model-version "$MODEL_VERSION" \
        --model-format "OpenAI" \
        --sku-capacity "$SKU_CAPACITY" \
        --sku-name "Standard" \
        --output none
    echo "    Created"
fi

# Read endpoint and key back from Azure — no manual copy-paste needed
echo "==> Reading credentials"
ENDPOINT=$(az cognitiveservices account show \
    --name "$RESOURCE_NAME" \
    --resource-group "$RESOURCE_GROUP" \
    --query "properties.endpoint" -o tsv)

API_KEY=$(az cognitiveservices account keys list \
    --name "$RESOURCE_NAME" \
    --resource-group "$RESOURCE_GROUP" \
    --query "key1" -o tsv)

cat > "$ENV_FILE" <<EOF
AZURE_OPENAI_ENDPOINT=$ENDPOINT
AZURE_OPENAI_API_KEY=$API_KEY
AZURE_OPENAI_DEPLOYMENT=$DEPLOYMENT_NAME
AZURE_OPENAI_API_VERSION=$API_VERSION
EOF

echo ""
echo ".env written to $ENV_FILE"
echo "Endpoint: $ENDPOINT"
```

---

## transcript_action_extractor.py

Main script. Reads a plain-text transcript file, calls Azure OpenAI with a schema-enforcing system prompt, and writes the extracted action items to a JSON file.

```python title="transcript_action_extractor.py" linenums="1"
import argparse
import json
import os
import sys
from pathlib import Path

from dotenv import load_dotenv
from openai import AzureOpenAI

load_dotenv(Path(__file__).resolve().parent / ".env")

ENDPOINT = os.environ["AZURE_OPENAI_ENDPOINT"]
API_KEY = os.environ["AZURE_OPENAI_API_KEY"]
DEPLOYMENT = os.environ["AZURE_OPENAI_DEPLOYMENT"]
API_VERSION = os.environ["AZURE_OPENAI_API_VERSION"]

_SYSTEM_PROMPT = """\
You are an assistant that extracts action items from meeting transcripts.

Return ONLY a valid JSON object with a single key "action_items" whose value is an array.
Each element must have exactly these fields:
  - "owner": the person responsible (string)
  - "task": what needs to be done (string)
  - "deadline": when it is due (string, or null if not stated)

Do not include any explanation or text outside the JSON object."""


def extract_action_items(transcript: str) -> list[dict]:
    if not transcript.strip():
        return []

    client = AzureOpenAI(
        azure_endpoint=ENDPOINT,
        api_key=API_KEY,
        api_version=API_VERSION,
    )

    response = client.chat.completions.create(
        model=DEPLOYMENT,
        response_format={"type": "json_object"},
        messages=[
            {"role": "system", "content": _SYSTEM_PROMPT},
            {"role": "user", "content": transcript},
        ],
    )

    data = json.loads(response.choices[0].message.content)
    return data.get("action_items", [])


def main() -> None:
    parser = argparse.ArgumentParser(
        description="Extract action items from a meeting transcript using Azure OpenAI."
    )
    parser.add_argument("transcript_file", help="Path to the plain-text transcript file")
    parser.add_argument(
        "--output",
        metavar="FILE",
        help="Write JSON output to FILE (default: <transcript_stem>.json in the same folder)",
    )
    args = parser.parse_args()

    transcript_path = Path(args.transcript_file).resolve()
    if not transcript_path.exists():
        print(f"Error: file not found: {transcript_path}", file=sys.stderr)
        sys.exit(1)

    transcript = transcript_path.read_text(encoding="utf-8")
    items = extract_action_items(transcript)
    output = json.dumps(items, indent=2, ensure_ascii=False)

    output_path = Path(args.output) if args.output else transcript_path.with_suffix(".json")
    output_path.write_text(output, encoding="utf-8")
    print(f"Wrote {len(items)} action item(s) to {output_path}")


if __name__ == "__main__":
    main()
```
