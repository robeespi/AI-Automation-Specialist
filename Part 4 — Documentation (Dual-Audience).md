## Part 4 — Documentation (Dual-Audience)

## 

## Part 4A — Client Documentation (Non-Technical)

### What This System Does

After each of your leadership or operations meetings, this system automatically produces a short, clear summary package — without you having to lift a finger.

**What you get within minutes of uploading a recording:**

* A one-page email summary with the key discussion points, three objectives, and clear action items with owners
* An editable PowerPoint slide deck (5 slides) ready for leadership distribution
* A copy of the cleaned transcript for your records

**The system never sends anything automatically.** Every output sits in a designated Google Drive folder and gets flagged in your team's Slack channel. A designated reviewer (typically Marcus) checks the summary, makes any edits, and then decides to send. The AI helps with the heavy lifting; your team stays in control.

\---

### How It Works, Step by Step

**What you do (2 minutes of effort):**

1. After a meeting ends, upload the recording file (from Google Meet or Zoom) to your designated Google Drive folder — "Meeting Inputs."
2. Optionally attach any relevant notes or spreadsheets from that meeting to the same folder.
3. That's it.

**What the system does automatically:**

1. Detects the new file and begins processing.
2. Converts the audio to a text transcript.
3. Reads the transcript and extracts the executive summary, top three objectives, action items (with owners and due dates), and next steps.
4. Builds a formatted slide deck and email draft.
5. Places both outputs in your "Meeting Outputs" folder in Google Drive.
6. Sends a Slack message to your leadership channel: "New meeting summary ready for review."

**What happens next (human step):**

* Marcus (or designated reviewer) opens the deck and email, checks for accuracy, makes any edits, and hits send.
* Nothing goes to the broader team without that approval.

\---

### What You Will Receive and When

|Output|Format|Typical Turnaround|
|-|-|-|
|Meeting transcript (cleaned)|.txt file in Google Drive|\~5 minutes after upload|
|Executive summary + action items|5-slide PowerPoint (editable)|\~8 minutes after upload|
|Email draft|Text in Google Drive + Slack alert|\~8 minutes after upload|
|Audit trail|JSON log in Drive|Automatically alongside outputs|

\---

### What the System Will NOT Do

* Send anything without a human approving it first
* Handle customer-facing communications (Phase 2, if you choose to expand)
* Make decisions or set strategy — it surfaces information to help you decide
* Process meetings that weren't recorded and uploaded

\---

### Benefits for BrightLane

**Time saved:** Marcus currently spends 15–20 hours per month on reporting and meeting follow-up. This system targets a 50%+ reduction — freeing that time for actual operational decisions.

**Better decisions:** Instead of compressed bullet points, leadership receives a summary that connects the dots — e.g., "Store 8 sales were down AND two staffing callouts occurred AND three managers flagged low stock in the bedding category." Context, not just data.

**Clear accountability:** Action items are extracted and assigned automatically. No more "I thought you were doing that."

**Cost:** Estimated at under $50/month — less than an hour of management time.

\---

\---

## Part 4B — Technical Documentation

### Architecture Overview

See the accompanying architecture diagram. The pipeline has 7 stages:

1. **Ingestion** — Audio/video file uploaded to Google Drive "Meeting Inputs" folder. Supporting documents (notes, spreadsheets) optionally co-uploaded.
2. **Trigger** — Google Drive webhook fires on file creation. n8n (or Make.com) workflow is invoked.
3. **Transcription** — Audio file downloaded, sent to OpenAI Whisper API. Transcript returned as text. Post-processing: speaker label normalisation, removal of filler words, chunking if >25 min.
4. **AI Inference** — Cleaned transcript passed to Claude API with a structured system prompt. Returns validated JSON containing: `executive\\\_summary`, `objectives\\\[3]`, `action\\\_items\\\[3]`, `next\\\_steps`, `risks\\\_and\\\_flags`, `confidence\\\_note`.
5. **Generation** — `python-pptx` generates the slide deck from JSON fields. Email draft rendered as HTML string. Both written to Drive "Meeting Outputs" folder.
6. **Human Review** — Slack notification posted to `#leadership-ops` with links to deck and email. Reviewer edits as needed, then manually sends email via Gmail.
7. **Audit** — Full JSON payload (transcript chunk, AI response, output file links, timestamp, model version) written to an audit log file in Google Cloud Storage or a designated Drive folder.

\---

### Tech Stack

|Layer|Technology|Why|
|-|-|-|
|Orchestration|n8n (self-hosted) or Make.com|Low-code, auditable, webhooks built-in|
|Transcription|OpenAI Whisper API|Best accuracy-to-cost ratio; handles retail jargon well|
|AI summarisation|Anthropic Claude (Sonnet 4)|Strong structured JSON output; low hallucination rate with proper prompting|
|Slide generation|python-pptx (Python)|Full programmatic control; editable .pptx output|
|Storage|Google Drive + GCS|Already in BrightLane's stack; familiar to ops team|
|Notifications|Slack API|Already used internally|
|Email delivery|Gmail API|Requires human to approve and send|

\---

### Environment Setup

```bash
# Python dependencies
pip install anthropic python-pptx google-api-python-client

# Environment variables (store in .env or secrets manager — NEVER commit to git)
ANTHROPIC\\\_API\\\_KEY=sk-ant-...
OPENAI\\\_API\\\_KEY=sk-...           # for Whisper transcription
GOOGLE\\\_DRIVE\\\_FOLDER\\\_INPUT=...   # Drive folder ID for uploads
GOOGLE\\\_DRIVE\\\_FOLDER\\\_OUTPUT=...  # Drive folder ID for outputs
SLACK\\\_WEBHOOK\\\_URL=https://hooks.slack.com/...
```

**API key management:**

* In development: `.env` file, loaded via `python-dotenv`
* In production: AWS Secrets Manager, GCP Secret Manager, or n8n's built-in credential store
* Keys are never logged, never included in outputs, never committed to version control
* Rotate keys quarterly or immediately if a breach is suspected

\---

### Prompt Engineering Notes

The Claude system prompt instructs the model to:

* Return only valid JSON (no markdown fences, no preamble)
* Extract exactly 3 objectives and 3 action items
* Ground all claims in the transcript — no hallucination
* Include a `confidence\\\_note` flagging areas of uncertainty
* Keep action items specific and operational (explicitly rejected: generic phrases like "improve communication")

**If output quality degrades:**

1. Check whether the transcript was cleanly formatted (filler words, crosstalk can confuse the model)
2. Review the `confidence\\\_note` field — it flags low-confidence extractions
3. Increase `max\\\_tokens` if responses are being truncated
4. Consider switching to Claude Opus for higher-stakes meetings

\---

### Failure Modes and Monitoring

|Failure|Detection|Response|
|-|-|-|
|Whisper transcription error|HTTP 4xx/5xx from API|Retry × 3 with backoff; alert Slack on final failure|
|Claude returns invalid JSON|`json.JSONDecodeError` caught|Log raw response to audit file; output fallback summary; alert reviewer|
|Drive upload fails|API error|Retry × 2; file saved locally as backup|
|Transcript too long (>100K tokens)|Character count check pre-API call|Chunk into 30-min segments; summarise each; merge summaries|
|Meeting audio is low quality|Whisper confidence score|Flag in output: "Transcript quality may be reduced — recommend human review of source"|

**Recommended monitoring:**

* n8n execution logs: review weekly, alert on any failed execution
* Audit JSON files: spot-check 2–3 per month for accuracy vs transcript
* Model version pinning: pin to a specific Claude and Whisper version in code; only upgrade after testing

\---

### Extending the System

**Adding a new output format (e.g., Google Slides instead of .pptx):**

* Replace the `python-pptx` step with the Google Slides API
* Same JSON input, different rendering function
* 2–4 hours of development

**Adding supporting document analysis (e.g., weekly sales spreadsheet):**

* Add a second Claude call with the spreadsheet data appended to the transcript context
* Prompt instructs model to cross-reference meeting discussion with data
* Adds \~$0.50–$1.00/month in token costs at baseline volume

**Phase 2 — Customer support triage:**

* Trigger: new ticket created in Gorgias
* Additional step: retrieve relevant policy docs from a knowledge base (RAG)
* Claude drafts response; routed to human review queue
* Auto-send only for well-defined, low-risk ticket categories (e.g., order status where tracking info is available)

\---

### Known Limitations of This POC

1. **No real Whisper integration** — the POC uses a transcript text file directly. Production requires the Whisper API call (stub provided in `poc\\\_workflow.py`).
2. **No Google Drive API integration** — files are read/written locally. Production requires Drive API auth and folder watching.
3. **Single-chunk processing** — transcripts >\~80K characters will hit token limits. Chunking logic is noted but not implemented.
4. **No retry logic** — the POC has a single API call with no retry on failure.
5. **Font availability** — `python-pptx` uses system fonts; on a cloud runner, specify font paths explicitly or use generic safe fonts (Arial, Calibri).

\---

*Prepared by: AI Automation Specialist Candidate
Date: April 2026
Assumptions made: meeting duration 45 min avg; Claude Sonnet 4 pricing as of April 2026; n8n Cloud Starter tier pricing; 20 meetings/month baseline*

