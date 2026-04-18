---
title: Paperless-ngx — Self-Hosted Document Management
description: Practical guide to deploying Paperless-ngx on a Synology NAS with Docker, OCR, multi-user access, and automated document classification.
date: 2026-04-18
tags:
  - self-hosted
  - docker
  - synology
  - nas
  - paperless-ngx
  - home-lab
status: published
---

# Paperless-ngx — Self-Hosted Document Management

A practical guide to deploying, organising, and running a self-hosted [Paperless-ngx](https://docs.paperless-ngx.com/) instance on a home NAS — covering setup, multi-user access, daily operations, and backup.

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
5. [Multi-User Setup](#5-multi-user-setup)
6. [Running It Day-to-Day](#6-running-it-day-to-day)
   - [6.1 Daily Routine](#61-daily-routine)
   - [6.2 Adding Documents](#62-adding-documents)
   - [6.3 Exception Handling](#63-exception-handling)
   - [6.4 Annual Tasks](#64-annual-tasks)
7. [Going Further](#7-going-further)
8. [Repository Contents](#8-repository-contents)

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

In practice, a non-technical second user can be onboarded with a single one-page guide covering VPN access, the daily inbox routine, and the tag system. The role-based permission model (§5) keeps family access simple without exposing system configuration.

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
| Language | `eng` | English OCR; multi-language supported (e.g. `eng+deu`, `eng+fra`) |
| Tesseract args | `--psm 1 --oem 3` | Automatic page layout detection; best available OCR engine |
| Barcode detection | Disabled | Skipped to reduce processing time |

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
- Each workflow has a **trigger** (e.g. document added), a **filter** (e.g. title contains a keyword), and **actions** (e.g. set type, add tag).
- Workflows run in priority order — specific rules first, a catch-all last.

**Recommended workflow set:**

| Priority | Purpose | Filter | Actions |
|---|---|---|---|
| 10 | Tax authority documents | Title contains tax authority name | Set Correspondent; Type: Government Notice; tag: `needs-review` |
| 20 | Bank statements | Title contains bank name | Set Correspondent; Type: Bank Statement; tag: `needs-review` |
| 30 | Payslips | Title contains employer name | Set Correspondent; Type: Payslip; tags: `income`, `needs-review` |
| 40 | Health / Medicare | Title contains health agency name | Set Correspondent; Type: Government Notice; tags: `medical-expense`, `needs-review` |
| 50 | Utility bills | Title contains provider names | Type: Invoice; tag: `needs-review` |
| 999 | **Catch-all** *(always last)* | *(none — matches everything)* | Tag: `needs-review` |

> **Priority tip:** Number specific workflows 10–50. Set the catch-all to 999. This guarantees specific rules always run before the fallback, and leaves room to insert new rules without renumbering.

---

## 5. Multi-User Setup

Paperless-ngx supports role-based access with group-level permissions. Using named groups rather than individual account flags keeps the access model explicit and auditable.

### Account and group structure

| Group | Who | Permissions |
|---|---|---|
| `Administrators` | Admin account | Full permissions on all objects — documents, taxonomy, settings, users |
| `Household` | Day-to-day user; family members | View and edit documents and taxonomy; no system configuration |
| `ReadOnly` | *(reserved)* | View only |

Two accounts are used in practice:

- **Admin account** — used only for system configuration, user management, and break-glass access. Keep the username non-obvious (not `admin`). Strong password distinct from the database password.
- **Power user account** — used for all daily document work. Member of Household group.

Additional family members get their own account in the Household group.

### Object-level permissions

In addition to group membership, Paperless-ngx requires object-level permissions on documents and taxonomy. When setting up:

1. Set owner of all existing documents, tags, correspondents, document types, and storage paths to the admin account
2. Grant **view** permission to: Administrators group + Household group
3. Grant **edit** permission to: Administrators group only

### Auto-assign permissions on consumption

Create a workflow that fires on every new document so family members can see newly ingested documents without manual intervention:

- **Trigger:** Document added
- **Filter:** *(none)*
- **Actions:** Set owner = admin account; grant view to Administrators + Household; grant edit to Administrators

Without this, documents consumed after the initial setup are invisible to non-admin users until permissions are applied manually — a common gotcha.

### Design rationale

A named `Administrators` group (rather than the raw superuser flag) makes the access model consistent and auditable — what the admin account can do is visible in the group permissions list, not implied by a hidden flag. The superuser flag remains available as a break-glass fallback.

---

## 6. Running It Day-to-Day

### 6.1 Daily Routine

Open the **Needs Review** saved view. Work through each document from top to bottom.

| Step | Action | Notes |
|---|---|---|
| 1 | Open the document | Check what workflows auto-assigned |
| 2 | Confirm or correct **Correspondent** | Fix if OCR misidentified the sender |
| 3 | Confirm or correct **Document Type** | Fix if the wrong type was assigned |
| 4 | Add **Financial Year tag** | e.g. `FY2026` for July 2025 – June 2026 |
| 5 | Add any **topic or tax tags** | e.g. `tax-deductible`, `income`, `medical-expense` |
| 6 | **Action required?** | If you need to pay, reply, or act — leave `needs-review` on and return once done |
| 7 | Remove `needs-review` | Document moves out of the inbox |

> Under 30 seconds per document once workflows are running. A full week's mail takes about 5 minutes. At tax time, everything is already tagged — just open the Tax saved view.

### 6.2 Adding Documents

**Desktop (Windows):** Map the consume folder as a network drive — `\\[your-nas-ip]\docker\paperless\consume`. Drag and drop PDFs directly in File Explorer. Requires home network or VPN when remote.

**Mobile:** Use the **DS File** app — navigate to `docker/paperless/consume/`, add it to Favourites, then tap + to upload. Works over QuickConnect without VPN.

**Phone scanning:** Scan paper documents with **Microsoft Lens** or **Adobe Scan** (or the iOS built-in document scanner). Save as PDF — not JPG — for better OCR results. Upload to the consume folder via DS File.

**Email:** Forward relevant emails to a dedicated Gmail address configured for IMAP ingestion. Paperless pulls attachments automatically and processes them like any other consumed file. Manual forwarding (rather than giving the address to senders directly) gives selective control — not every email from a given sender is worth archiving.

| Source | Method |
|---|---|
| Paper | Scan with phone → save as PDF → upload via DS File or SMB |
| Digital PDF (email, download) | Save to consume folder via SMB or DS File |
| Screenshot / image | JPG or PNG to consume — Paperless OCRs images as well as PDFs |
| Email attachment | Forward to dedicated Gmail address (IMAP ingestion) |

> **Naming tip:** Rename files descriptively before dropping them in — e.g. `bank-statement-mar-2026.pdf`. The filename becomes the initial document title, which workflow filters match against.

### 6.3 Exception Handling

| Problem | Symptom | Fix |
|---|---|---|
| **Duplicate document** | Same document ingested twice | Search by correspondent or phrase before classifying; delete the newer copy if confirmed duplicate |
| **Failed OCR** | Document has no searchable text | Check Settings → Tasks for errors. Re-scan at 400 DPI minimum, or remove password protection. If re-scanning is not possible, set Title / Type / Correspondent manually — the document is stored regardless |
| **Workflow didn't fire** | Document in Needs Review with no auto-assigned fields | OCR likely misread the title. Correct the title manually → re-save → the catch-all workflow fires and moves the document to `filed` automatically |
| **Garbled OCR / wrong language** | Text is meaningless | English-only OCR — foreign-language documents store safely but text won't be searchable. Set metadata manually as a workaround |

### 6.4 Annual Tasks

Do these every **July** at the start of the new financial year:

| Task | Where | Details |
|---|---|---|
| Add new FY tag | Settings → Tags → Add | e.g. `FY2027` each July |
| Create Tax saved views for new year | Documents → Filter → Save | Tax — FY2027 (tag: FY2027 + tax-deductible); Tax — Income FY2027 (tag: FY2027 + income) |
| Clear the inbox | Needs Review view | Classify or action any documents left over from the previous year |
| Review `filed` documents | Optional | Archive documents older than 5 years by adding the `archive` tag |

---

## 7. Going Further

### Security hardening

These steps should be done before the instance is in regular use:

| Step | What to do |
|---|---|
| PostgreSQL password | Run via Container Manager → db → Terminal: `ALTER USER paperless WITH PASSWORD '[strong-password]';` |
| Secret key | Generate: `openssl rand -base64 37 \| tr -d '=+/' \| cut -c1-50` → set result as `PAPERLESS_SECRET_KEY` env var on the webserver container |
| Allowed hosts | Set `PAPERLESS_ALLOWED_HOSTS` to `[your-nas-ip],localhost,127.0.0.1,[your-ddns-hostname]`. Django returns HTTP 400 if the Host header is not on this list — the DDNS hostname must be included if you access Paperless by hostname. |
| Admin account | Rename the default `admin` account to a non-obvious username. Use a strong password distinct from the database password. |

**Remote access:** For a private home deployment, VPN-only access is the practical choice — no extra open port, no TLS certificate to manage, and the VPN tunnel handles transport security. Connect via OpenVPN, then access `http://[your-nas-ip]:8000/` or the DDNS hostname from any browser.

For deployments that need internet-facing access without VPN, a reverse proxy (e.g. Nginx, Traefik, or Synology Reverse Proxy) with a Let's Encrypt certificate is the standard path.

### Automated backups

A two-layer approach covers both full recovery and document-level recovery:

| Layer | What it covers | How |
|---|---|---|
| **Volume backup** | Everything — database, documents, config, Redis state | Back up entire `/volume1/docker/` to an external USB or NAS backup destination. Schedule: three times a week (e.g. Mon/Thu/Sat) via Hyper Backup |
| **Document export** | All documents and metadata in a portable, human-readable format | DSM Task Scheduler: daily at 01:00. Script: `docker exec webserver document_exporter ../export`. Output: `/volume1/docker/paperless/export/` |

The export produces a `manifest.json` + `documents/` folder that can be imported into a fresh Paperless instance independently of the volume backup.

**What the document export does NOT include** — these must be re-created manually after a fresh-install restore:

- User accounts and passwords
- Saved views
- Mail rules and email ingestion config
- Automation workflows
- Environment variables (`PAPERLESS_SECRET_KEY`, `PAPERLESS_ALLOWED_HOSTS`)

### Restore

| Scenario | When to use | Approach |
|---|---|---|
| **Add missing documents** | Database intact, some documents missing | `docker exec webserver document_importer ../export` |
| **Full wipe and reimport** | Corrupted data, fresh database needed | Stop webserver → drop and recreate the PostgreSQL schema → start webserver → wait 30 s → reimport → recreate superuser |
| **Disaster recovery** (new NAS) | Total hardware loss | Restore `/volume1/docker/` from volume backup → recreate container stack → reimport from export if pgdata is unusable |

After any restore: verify documents are visible, tags and correspondents are present, the consume folder is being monitored, and backup jobs are re-enabled.

### Email ingestion

Paperless-ngx can pull email attachments via IMAP and process them exactly like files dropped into the consume folder.

**Gmail setup:**

| Setting | Value |
|---|---|
| Dedicated address | Create a separate Gmail account (e.g. `[yourname]-paperless@gmail.com`) |
| App password | Google Account → Security → 2-Step Verification → App passwords (16-character) |
| IMAP server | `imap.gmail.com`, port `993`, SSL/TLS |

**Mail rule (catch-all):**

| Setting | Value |
|---|---|
| Order | 999 |
| Action | Consume attachments |
| Assign tags | `needs-review` |
| After processing | Mark email as read |

Forward relevant emails to the dedicated address rather than giving it directly to senders — this keeps selective control over what gets archived. Because Gmail rewrites the `From:` header on forwarded mail, per-sender filter rules don't work reliably; a single catch-all rule handles everything, with document workflows doing classification after OCR.

### Optional enhancements

| Feature | What it adds |
|---|---|
| **Storage Paths** | Auto-organise archived files into a folder structure — e.g. `{correspondent}/{document_type}/{created_year}/` produces `ATO/Government Notice/2025/[title].pdf` |
| **Custom Fields** | Add structured metadata to documents: Amount, Account Number, Policy Number, Expiry Date, Reference Number |
| **Share Links** | Generate temporary read-only links to individual documents — no account required for the recipient |
| **API integration** | Use the REST API to trigger ingestion, query documents, or integrate with home automation tools |

### Ongoing maintenance

| Task | Frequency |
|---|---|
| Add new financial year tag and saved views | Annually (start of each fiscal year) |
| Update container images | Every 2–3 months |
| Check task queue for failed jobs | Monthly |
| Verify backups are completing successfully | Monthly |
| Annual restore test — confirm export imports cleanly to a fresh instance | Annually |
| Review and archive old status-tagged documents | Quarterly |

---

## 8. Repository Contents

This repository contains deployment documentation and configuration references for the above setup. It is not the Paperless-ngx source code.

| File | Description |
|---|---|
| `Paperless-ngx_Complete_Reference/` | Full deployment reference split across 8 files: architecture, containers, volumes, environment variables, OCR settings, users and permissions, backup and restore, and quick reference |
| `paperless-ngx-organisation-guide.md` | Complete taxonomy guide — document types, correspondents, tags, saved views, workflows, and day-to-day process |
| `paperless-ngx-day-to-day-sop.md` | Day-to-day SOP — inbox routine, scanning, bulk import, exception handling, phone workflow, and annual tasks |
| `paperless-ngx-next-steps.md` | Actionable checklist — UI setup tasks, security improvements, backup automation, and optional enhancements |
| `Paperless-ngx_Restore_Procedure.md` | Step-by-step restore procedures: add missing documents and full wipe + reimport |
| `rbac-design.md` | RBAC design — accounts, groups, object-level permissions, and implementation steps |
| `gabi-onboarding-guide.md` | Non-technical quick guide for household users — access, daily routine, classification, and quick reference |
| `Paperless-ngx.md` | Original per-container setup notes for Synology Container Manager GUI |
| `OCR Settings.md` | OCR parameter reference with descriptions |
| `Paperlessngx - Poweruser.md` | Power user permission template — full CRUD matrix |
| `Paperlessngx-FolderCheckScript.md` | Bash script to verify folder structure and container volume mounts |

---

*Built with [Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx) — open-source document management.*
