---
title: "AI Knows What Was Said. This Script Knows What to Do About It."
description: A lightweight Python tool that skips the meeting summary and goes straight to structured, owner-attributed action items in JSON.
date: 2026-04-24
tags:
  - python
  - azure
  - azure-openai
  - ai
  - project-management
  - automation
  - workplace
status: published
---

Your AI meeting tool just summarised a 45-minute call into four paragraphs of fluent prose. Useful, if what you needed was a document. But you're a project manager. What you actually needed was: who owns what, and by when.

Those are not the same thing.

---

## The Problem With Meeting AI

Most tools in this space — Copilot, Otter, Fireflies, and the rest — are optimised for *narration*. They retell the meeting. The output reads like a well-written summary, which is genuinely impressive, and almost entirely useless for tracking work.

A PM still has to read through the prose, decide what counts as a commitment, figure out who said it, guess whether a deadline was implied, and then manually paste everything into Jira or Monday. The AI handled the easy part and left you with the actual work.

The gap is not a quality problem — the summaries are good. It's a *format* problem. Prose is designed for humans to read once. Action items are data. They need to be stored, queried, assigned, and integrated with the tools your team already uses.

---

## The Fix: Structured Output, Not a Summary

This is a ~80-line Python script that does one thing: reads a meeting transcript and returns a JSON array of action items, each with exactly three fields.

```json
[
  {
    "owner": "James",
    "task": "Complete pagination changes on the front end for the search refactor",
    "deadline": "2026-04-30"
  },
  {
    "owner": "Priya",
    "task": "Set up new regression test cases for the search feature",
    "deadline": "2026-04-29"
  },
  {
    "owner": "Tom",
    "task": "Rotate the staging API keys",
    "deadline": "2026-04-27"
  },
  {
    "owner": "Sarah",
    "task": "Send the updated roadmap deck to the stakeholders",
    "deadline": "2026-04-24"
  }
]
```

No narrative. No interpretation required. Every item has an owner, a task, and a deadline — or `null` if one wasn't mentioned. The output is machine-readable by design, ready to be piped into whatever comes next.

The model behind it is gpt-4o on Azure OpenAI, but the approach is not OpenAI-specific — any model available through Azure OpenAI works. Swap the deployment name, keep the rest.

---

## How It Works

The script is built around a single importable function:

```python title="transcript_action_extractor.py" linenums="1"
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
```

Two things worth noting:

**The system prompt enforces the schema.** It doesn't ask the model to "try to" return JSON — it instructs it to return *only* a JSON object with a specific structure, no explanation, nothing else.

**`response_format={"type": "json_object"}` backs that up at the API level.** This is structured output enforcement, not just prompting. The model is constrained to return valid JSON regardless of what the transcript throws at it.

The function is importable, so it can be called directly from other scripts — useful if this becomes one stage in a larger pipeline.

[:material-code-braces: Full source code](../../code/transcript-action-extractor.md){ .md-button }

---

## Setup

**Prerequisites:**

- Python 3.11+
- Azure CLI installed and authenticated (`az login`)
- An active Azure subscription with available Azure OpenAI quota

**Install dependencies:**

```bash
pip install openai python-dotenv
```

**Provision the Azure stack:**

`setup.sh` handles the entire Azure side: it creates a resource group, provisions an Azure OpenAI resource, deploys gpt-4o, reads the endpoint and key back from Azure, and writes them to a `.env` file — no portal, no copy-paste.

```bash
bash setup.sh
```

The script is idempotent. If the resource already exists, it skips creation and moves on.

!!! note "Region availability"
    The script targets `australiaeast` by default. If you have gpt-4o quota in another region (e.g. `eastus`, `swedencentral`), update the `LOCATION` variable at the top of `setup.sh` before running.

After `setup.sh` completes, a `.env` file is written next to the script with four variables:

```
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_API_KEY=...
AZURE_OPENAI_DEPLOYMENT=transcript
AZURE_OPENAI_API_VERSION=2024-06-01
```

---

## Running It

Point the script at any plain-text transcript file:

```bash
python transcript_action_extractor.py meeting-transcript-1.txt
```

Output is written to `meeting-transcript-1.json` in the same folder. To specify a different path:

```bash
python transcript_action_extractor.py meeting-transcript-1.txt --output actions.json
```

The script prints a confirmation to stdout:

```
Wrote 5 action item(s) to meeting-transcript-1.json
```

The function handles transcripts with no action items gracefully — it returns an empty array without crashing.

---

## What You Can Do With This

JSON is the lingua franca of integrations. Once your action items are in that format, the next step is usually one API call:

- **Jira** — POST to the issues endpoint, map `owner` to assignee, `task` to summary, `deadline` to due date
- **Monday.com** — GraphQL mutation to create items in a board
- **Microsoft Teams** — Adaptive Card per item, sent to the relevant channel after the meeting
- **Spreadsheet / database** — append rows directly, no parsing required

The script was also designed to slot into a larger pipeline: a meeting audio → transcript → structured notes workflow where this function handles the final extraction stage. The importable `extract_action_items` signature makes that straightforward.

---

## Key Takeaways

- Meeting AI that outputs prose still puts the extraction work on you — structured output removes it
- Three fields (`owner`, `task`, `deadline`) are enough to make an action item useful and automatable
- `response_format={"type": "json_object"}` enforces schema at the API level, not just through prompting
- `setup.sh` provisions the full Azure OpenAI stack in one command — no manual portal steps
- The model is swappable; the pattern is not OpenAI-specific
