---
title: "TTS Audio Generator — Source Code"
description: "Azure infrastructure setup script and Python TTS generator for speaker-labelled transcripts."
date: 2026-05-07
tags:
  - python
  - azure
  - text-to-speech
  - automation
  - workplace
status: published
---

# TTS Audio Generator — Source Code

Source files for the [Meeting Audio Notes Pipeline](../projects/meeting-audio-notes-pipeline.md) — a companion tool that converts speaker-labelled transcripts to MP3 using Azure AI Speech TTS with gender-matched neural voices per speaker.

---

## setup.sh

Provisions an Azure AI Speech resource and writes `AZURE_SPEECH_KEY` and `AZURE_SPEECH_REGION` to `.env`. The pipeline's own `setup.sh` provisions the same resource — if it already exists the script skips creation safely.

```bash title="setup.sh" linenums="1"
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
ENV_FILE="$SCRIPT_DIR/.env"

RESOURCE_GROUP="rg-fn-audio-pipeline"
SPEECH_RESOURCE="speech-fn-audio-pipeline"
SPEECH_LOCATION="eastus"

if ! command -v az &>/dev/null; then
    echo "Error: Azure CLI not found." >&2
    echo "Install from https://docs.microsoft.com/cli/azure/install-azure-cli" >&2
    exit 1
fi

if ! az account show &>/dev/null 2>&1; then
    echo "Error: not logged in to Azure. Run: az login" >&2
    exit 1
fi

echo "Subscription: $(az account show --query '[name, id]' -o tsv | tr '\t' ' / ')"
echo ""

echo "==> Resource group: $RESOURCE_GROUP"
az group create --name "$RESOURCE_GROUP" --location "$SPEECH_LOCATION" --output none
echo "    OK"

echo "==> Azure AI Speech: $SPEECH_RESOURCE ($SPEECH_LOCATION)"
if az cognitiveservices account show \
        --name "$SPEECH_RESOURCE" --resource-group "$RESOURCE_GROUP" &>/dev/null 2>&1; then
    echo "    Already exists, skipping"
else
    az cognitiveservices account create \
        --name "$SPEECH_RESOURCE" \
        --resource-group "$RESOURCE_GROUP" \
        --kind "SpeechServices" \
        --sku "S0" \
        --location "$SPEECH_LOCATION" \
        --yes \
        --output none
    echo "    Created"
fi

echo "==> Reading credentials"
SPEECH_KEY=$(az cognitiveservices account keys list \
    --name "$SPEECH_RESOURCE" \
    --resource-group "$RESOURCE_GROUP" \
    --query "key1" -o tsv)

cat > "$ENV_FILE" <<EOF
AZURE_SPEECH_KEY=$SPEECH_KEY
AZURE_SPEECH_REGION=$SPEECH_LOCATION
EOF

echo ""
echo ".env written to $ENV_FILE"
```

---

## tts.py

Reads a speaker-labelled transcript (`Speaker: line` format), assigns `en-US-JennyNeural` or `en-US-GuyNeural` per speaker based on name, builds SSML with per-speaker `<voice>` tags, and synthesises the full audio to MP3 via the Azure Speech SDK.

Usage: `python tts.py <transcript.txt> [--output <name.mp3>]`

```python title="tts.py" linenums="1"
"""Convert a speaker-labelled transcript to MP3 using Azure AI Speech TTS.

Each speaker is automatically assigned a feminine or masculine neural voice based
on their name. Add names to FEMALE_NAMES / MALE_NAMES to extend coverage.
"""

import argparse
import os
import re
import sys
from pathlib import Path

import azure.cognitiveservices.speech as speechsdk
from dotenv import load_dotenv

load_dotenv()

FEMALE_VOICE = "en-US-JennyNeural"
MALE_VOICE = "en-US-GuyNeural"

FEMALE_NAMES = {
    "sarah", "priya", "aria", "emily", "emma", "lisa", "anna", "kate",
    "jessica", "linda", "sophia", "olivia", "ava", "mia", "natalie",
}
MALE_NAMES = {
    "james", "tom", "david", "john", "michael", "robert", "chris", "daniel",
    "matt", "alex", "ryan", "adam", "brian", "kevin", "mark", "steve",
}

_SPEAKER_RE = re.compile(r"^([A-Za-z][A-Za-z\s]*):\s*(.*)")

_SSML_WRAP = (
    '<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">'
    "{body}"
    "</speak>"
)


def _voice(name: str) -> str:
    n = name.strip().lower()
    if n in FEMALE_NAMES:
        return FEMALE_VOICE
    if n in MALE_NAMES:
        return MALE_VOICE
    return MALE_VOICE  # default fallback


def _xml(text: str) -> str:
    return (
        text.replace("&", "&amp;")
            .replace("<", "&lt;")
            .replace(">", "&gt;")
    )


def build_ssml(text: str) -> str:
    lines = text.splitlines()
    blocks: list[str] = []
    past_divider = False
    current_speaker: str | None = None
    current_lines: list[str] = []

    def flush() -> None:
        if not current_lines:
            return
        content = _xml(" ".join(l for l in current_lines if l).strip())
        if not content:
            return
        voice = _voice(current_speaker) if current_speaker else MALE_VOICE
        blocks.append(f'<voice name="{voice}">{content}</voice>')
        current_lines.clear()

    for line in lines:
        stripped = line.strip()

        if stripped == "---":
            if not past_divider:
                flush()
            past_divider = True
            continue

        if not past_divider:
            # Header section — read as a single block in the default voice
            if stripped:
                current_lines.append(stripped)
            continue

        m = _SPEAKER_RE.match(line)
        if m:
            flush()
            current_speaker = m.group(1).strip()
            tail = m.group(2).strip()
            if tail:
                current_lines.append(tail)
        elif stripped:
            current_lines.append(stripped)

    flush()
    return _SSML_WRAP.format(body="".join(blocks))


def synthesize(text_path: Path, output_path: Path) -> None:
    key = os.environ.get("AZURE_SPEECH_KEY")
    region = os.environ.get("AZURE_SPEECH_REGION")
    if not key or not region:
        print("Error: AZURE_SPEECH_KEY and AZURE_SPEECH_REGION must be set in .env", file=sys.stderr)
        sys.exit(1)

    config = speechsdk.SpeechConfig(subscription=key, region=region)
    config.set_speech_synthesis_output_format(
        speechsdk.SpeechSynthesisOutputFormat.Audio16Khz32KBitRateMonoMp3
    )

    audio_config = speechsdk.audio.AudioOutputConfig(filename=str(output_path))
    synthesizer = speechsdk.SpeechSynthesizer(speech_config=config, audio_config=audio_config)

    text = text_path.read_text(encoding="utf-8")
    ssml = build_ssml(text)

    print(f"Synthesising {len(text)} characters → {output_path.name} ...")
    result = synthesizer.speak_ssml_async(ssml).get()

    if result.reason == speechsdk.ResultReason.SynthesizingAudioCompleted:
        print(f"Done: {output_path}")
    elif result.reason == speechsdk.ResultReason.Canceled:
        details = result.cancellation_details
        print(f"Error: {details.reason}", file=sys.stderr)
        if details.error_details:
            print(f"       {details.error_details}", file=sys.stderr)
        sys.exit(1)


def main() -> None:
    parser = argparse.ArgumentParser(
        description="Convert a speaker-labelled transcript to MP3 via Azure AI Speech TTS"
    )
    parser.add_argument("input", type=Path, help="Path to the .txt transcript file")
    parser.add_argument("--output", type=Path, help="Output .mp3 path (default: same name as input)")
    args = parser.parse_args()

    if not args.input.is_file():
        print(f"Error: input file not found: {args.input}", file=sys.stderr)
        sys.exit(1)

    output = args.output or args.input.with_suffix(".mp3")
    synthesize(args.input, output)


if __name__ == "__main__":
    main()
```
