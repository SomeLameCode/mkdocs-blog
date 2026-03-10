# Paperless-ngx — Self-Hosted Document Management

A practical guide to deploying, organising, and getting the most out of a self-hosted [Paperless-ngx](https://docs.paperless-ngx.com/) instance on a home NAS.

---

## Table of Contents

1. [What Is Paperless-ngx?](#1-what-is-paperless-ngx)
2. [Who Is It For?](#2-who-is-it-for)
3. [Solution Architecture](#3-solution-architecture)
   - [3.1 The Five-Service Stack](#31-the-five-service-stack)
   - [3.2 Document Processing Pipeline](#32-document-processing-pipeline)
   - [3.3 OCR Configuration](#33-ocr-configuration)
4. [Document Organisation System](#4-document-organisation-system)
   - [4.1 The Three Classification Tools](#41-the-three-classification-tools)
   - [4.2 Document Types](#42-document-types)
   - [4.3 Tags](#43-tags)
   - [4.4 Saved Views](#44-saved-views)
   - [4.5 Automated Workflows](#45-automated-workflows)
5. [What Else Can Be Done](#5-what-else-can-be-done)
6. [Repository Contents](#6-repository-contents)

---

## 1. What Is Paperless-ngx?

**Paperless-ngx** is a free, open-source document management system you host yourself. It ingests scanned documents and PDFs, runs OCR to make them fully searchable, classifies them with metadata, and archives everything in a structured, browsable web interface.

> **Key idea:** You scan or drop a document into a folder. Paperless-ngx handles the rest — text recognition, archiving, indexing — and makes it searchable in seconds.

### Core capabilities

| Capability | Description |
|---|---|
| OCR | Converts scans and image-based PDFs into searchable text using Tesseract |
| Full-text search | Find any document by content, not just filename |
| Classification | Organise documents with types, correspondents, tags, and custom fields |
| Automation | Auto-classify incoming documents using configurable workflow rules |
| Archive | Produces standardised PDF/A output alongside original files |
| Web UI | Clean, responsive interface accessible from any browser |
| API | Full REST API for integration with other tools and scripts |
| Multi-user | Role-based access with per-user saved views and permissions |

---

## 2. Who Is It For?

Paperless-ngx is designed for anyone who deals with a recurring flow of documents and wants to stop losing things in folders or filing cabinets.

### Everyday household users

Most households accumulate hundreds of documents per year — bank statements, insurance policies, tax records, medical letters, utility bills. Paperless-ngx solves the common problems:

- **"I can't find that document."** — Full-text search across everything, instantly.
- **"I'm not sure if I kept that."** — Automatic ingestion means nothing gets lost.
- **"Tax time is a nightmare."** — Tag documents as they arrive; at tax time, open a saved view.
- **"I have paper everywhere."** — Scan once, discard the paper, find it digitally forever.

### Small home offices and freelancers

- Centralised storage for invoices, contracts, receipts, and client correspondence
- Search by client name, document type, or date range
- Export documents for accountants or legal review

### Families managing shared documents

- Multiple user accounts with separate saved views
- Shared correspondents and document types
- Personal health records, identity documents, and property papers in one searchable archive

> **Privacy note:** Because Paperless-ngx is self-hosted, your documents never leave your own hardware. No third-party cloud service has access to your files.

---

## 3. Solution Architecture

This deployment runs Paperless-ngx on a **Synology NAS** using Docker containers managed through Synology Container Manager. The same stack can run on any Linux host with Docker.

### 3.1 The Five-Service Stack

```
                    ┌─────────────────────────────────────────┐
                    │         paperless-net (bridge)           │
                    │                                          │
  [browser] :8000 ──►  webserver (paperless-ngx)              │
                    │      │          │          │             │
                    │   broker      db (pg)   gotenberg        │
                    │  (redis)    port 5432   port 3000        │
                    │                           │              │
                    │                         tika             │
                    │                        port 9998         │
                    └─────────────────────────────────────────┘
```

| Container | Image | Role |
|---|---|---|
| `webserver` | `ghcr.io/paperless-ngx/paperless-ngx:latest` | Django web application — the main interface and task runner |
| `db` | `postgres:17` | Stores all document metadata, tags, users, and classification data |
| `broker` | `redis:8` | Task queue — coordinates background OCR and ingestion jobs |
| `gotenberg` | `gotenberg/gotenberg:8.20` | Converts non-PDF formats (Word, HTML, etc.) to PDF |
| `tika` | `apache/tika:latest` | Extracts content and metadata from complex file formats |

All containers communicate over a single user-defined Docker bridge network. Only the `webserver` exposes a port (`8000`) to the host.

### 3.2 Document Processing Pipeline

When a file is dropped into the consume folder, the following happens automatically:

```
Drop file into consume/
        │
        ▼
  Format detection
  (Tika + Gotenberg)
        │
        ▼
  OCR processing
  (Tesseract via paperless)
        │
        ▼
  Workflow rules evaluated
  (auto-assign type, correspondent, tags)
        │
        ▼
  Archived PDF/A created
  Original file preserved
        │
        ▼
  Full-text index updated
  Document visible in UI
```

### 3.3 OCR Configuration

The OCR pipeline is configured for high-quality archival output:

| Setting | Value | Effect |
|---|---|---|
| Output format | PDF/A-2b | Long-term archival standard |
| OCR mode | Force | Processes all pages even if text layer exists |
| Image DPI | 400 | High-resolution processing for accurate recognition |
| Cleaning | clean-final | Applies image correction before OCR |
| Deskew | Enabled | Corrects tilted scans automatically |
| Auto-rotate | Enabled | Corrects rotated pages automatically |
| Colour conversion | Grayscale | Improves contrast; reduces file size |
| Archive files | Always | Originals are preserved alongside archived copies |
| Language | Configurable | Multi-language OCR supported (e.g. `eng`, `eng+deu`, `eng+fra`) |

---

## 4. Document Organisation System

Paperless-ngx provides three classification tools. Used together, they make any document findable in seconds without relying on folder structure or filename conventions.

### 4.1 The Three Classification Tools

| Tool | Question it answers | Cardinality |
|---|---|---|
| **Document Type** | What kind of document is it? | One per document |
| **Correspondent** | Who sent it, or who is it about? | One per document |
| **Tags** | What topics does it relate to? | Many per document |

> **Key principle:** Document Type and Correspondent narrow down *what* and *who*. Tags handle everything else — year, topic, tax relevance, processing status. Use tags generously.

### 4.2 Document Types

Document types are broad, stable categories. The goal is a short list that covers all document varieties without becoming granular.

**Recommended categories:**

| Type | Covers |
|---|---|
| Bank Statement | Monthly or quarterly statements from financial institutions |
| Invoice | Bills received — utilities, services, subscriptions |
| Receipt | Proof of payment — purchases, donations, expenses |
| Payslip | Salary and wage records |
| Tax Document | Returns, assessments, income statements |
| Contract | Signed agreements — employment, rental, services |
| Insurance Policy | Policies, certificates of currency, renewals |
| Medical Record | Test results, referrals, discharge summaries |
| Government Notice | Official correspondence from government agencies |
| Property Document | Titles, rates notices, inspection reports, leases |
| Correspondence | General letters and emails not covered above |
| Identity Document | Passports, licences, birth certificates, visas |

### 4.3 Tags

Tags are the most flexible part of the system. A single document can carry many tags, making cross-cutting searches possible.

**Recommended tag groups:**

**Financial year tags** — one per document to enable year-based filtering:
`FY2024`, `FY2025`, `FY2026` *(add a new one each July)*

**Tax tags** — applied throughout the year so nothing needs to be found at tax time:
`tax-deductible`, `income`, `capital-gain`, `donation`, `work-expense`, `home-office`, `vehicle`, `private-health`, `tax-return`

**Topic tags** — for personal finance, property, and health:
`superannuation`, `investment`, `mortgage`, `property-home`, `medical-expense`, `prescription`, `dental`

**Status tags** — for managing the processing pipeline:
`needs-review`, `important`, `pending`, `filed`, `archive`

> **Tip:** Apply `needs-review` automatically via a catch-all workflow. This creates a reliable inbox of documents that need human attention before being considered filed.

### 4.4 Saved Views

Saved Views are pre-configured filtered lists that act as one-click shortcuts. Create them once; use them permanently.

**Recommended views:**

| View | Purpose |
|---|---|
| Needs Review | All documents awaiting classification confirmation |
| Tax — Current FY | All deductible expenses for the current financial year |
| Tax — Income Current FY | All income documents for the current year |
| Recent Documents | Everything added in the last 30 days |
| Pending Actions | Documents tagged `pending` — awaiting a response or follow-up |
| Current Insurance | Active insurance policies |
| Medical Records | All health-related documents |

### 4.5 Automated Workflows

Workflows automatically apply classification rules when a document is ingested. They eliminate most manual classification work.

**How they work:**
- Each workflow has a **trigger** (e.g. document added), a **filter** (e.g. title contains "invoice"), and **actions** (e.g. set type = Invoice, add tag = `needs-review`).
- Workflows run in priority order — specific rules first, a catch-all last.

**Recommended workflow set:**

| Workflow | Match condition | Actions |
|---|---|---|
| Agency / Tax Authority | Title contains tax authority name | Set correspondent, type = Government Notice |
| Bank Statements | Title contains bank name | Type = Bank Statement |
| Payslips | Title contains employer name | Type = Payslip, tag = `income` |
| Healthcare | Title contains health agency name | Tag = `medical-expense` |
| Utility Bills | Title contains provider name | Type = Invoice |
| **Catch-all** *(priority 999)* | All documents | Tag = `needs-review` |

> **Priority tip:** Number specific workflows 10–50. Set the catch-all to 999. This guarantees specific rules always run before the fallback.

---

## 5. What Else Can Be Done

The core deployment covers ingestion, OCR, and classification. The following additions improve security, resilience, and capability.

### Security hardening

| Improvement | Description |
|---|---|
| Change default credentials | Replace placeholder database passwords with strong unique values |
| Add a secret key | Set `PAPERLESS_SECRET_KEY` to a random string for secure session signing |
| Restrict allowed hosts | Set `PAPERLESS_ALLOWED_HOSTS` to prevent host header injection |
| Enable HTTPS | Route traffic through a reverse proxy (e.g. Nginx, Traefik, or Synology built-in) with a valid TLS certificate |

### Automated backups

| Method | Description |
|---|---|
| Scheduled export | Use a scheduled task to run `document_exporter` weekly — produces a portable, human-readable backup |
| Volume backup | Back up the database (`pgdata/`) and document store (`media/`) using your NAS backup tool or rsync |
| Off-site copy | Sync the export folder to cloud storage (S3, Backblaze B2, OneDrive) for offsite redundancy |

### Optional enhancements

| Feature | What it adds |
|---|---|
| **Storage Paths** | Auto-organise archived files into a folder structure (e.g. by correspondent / type / year) |
| **Email ingestion** | Pull documents directly from an IMAP inbox — useful for digital invoices and statements |
| **Custom Fields** | Add structured metadata: amount, account number, expiry date, policy number |
| **Share Links** | Generate temporary read-only links to individual documents — no account required for the recipient |
| **Remote access** | Combine a reverse proxy with a dynamic DNS hostname and Let's Encrypt certificate for secure access from anywhere |
| **API integration** | Use the REST API to trigger ingestion, query documents, or integrate with home automation tools |

### Ongoing maintenance

| Task | Frequency |
|---|---|
| Add new financial year tag and saved views | Annually (start of each fiscal year) |
| Update container images | Every 2–3 months |
| Check task queue for failed jobs | Monthly |
| Verify backups are completing successfully | Monthly |
| Review and archive old status-tagged documents | Quarterly |

---

## 6. Repository Contents

This repository contains deployment documentation and configuration references for the above setup. It is not the Paperless-ngx source code.

| File | Description |
|---|---|
| `Paperless-ngx_Complete_Reference.md` | Full deployment reference — all containers, volumes, environment variables, OCR settings, and backup/restore procedure |
| `paperless-ngx-organisation-guide.md` | Complete taxonomy guide — document types, correspondents, tags, saved views, workflows, and day-to-day process |
| `paperless-ngx-next-steps.md` | Actionable checklist — UI setup tasks, security improvements, backup automation, and optional enhancements |
| `Paperless-ngx.md` | Original per-container setup notes for Synology Container Manager GUI |
| `OCR Settings.md` | OCR parameter reference with descriptions |
| `Paperlessngx - Poweruser.md` | Power user permission template — full CRUD matrix |
| `Paperlessngx-FolderCheckScript.md` | Bash script to verify folder structure and container volume mounts |

---

*Built with [Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx) — open-source document management.*
