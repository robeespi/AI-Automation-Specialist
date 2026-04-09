## Part 2 — Technology Cost Estimate

### Assumptions

* 20–25 meetings/month, average 45 minutes each
* \~1 hour of audio = \~25,000 tokens of transcript text
* Claude Sonnet used for summarisation (cost-effective; Opus available for higher quality)
* Google Workspace already licensed — no additional cost allocated
* n8n (self-hosted on a small cloud VM) used as orchestrator
* Storage costs based on 90-day retention of transcripts, decks, and audit logs

\---

### Monthly Cost Table (20 meetings/month baseline)

|Component|Service|Unit Cost|Monthly Usage|Monthly Cost|
|-|-|-|-|-|
|Audio transcription|OpenAI Whisper API|$0.006 / min|900 min (20×45)|\~$5.40|
|AI summarisation (input)|Claude Sonnet 4|$3.00 / 1M tokens|\~500K tokens|\~$1.50|
|AI summarisation (output)|Claude Sonnet 4|$15.00 / 1M tokens|\~100K tokens|\~$1.50|
|Workflow orchestration|n8n Cloud (Starter)|$20/month flat|—|$20.00|
|Cloud VM (n8n self-hosted alt.)|AWS t3.small|\~$15/month|—|$15.00|
|File storage|Google Cloud Storage|$0.02 / GB|\~5 GB/month|\~$0.10|
|Email delivery|Gmail API (Google Workspace)|Included|—|$0.00|
|Slack notifications|Slack API|Free tier|—|$0.00|
|Google Drive output upload|Drive API|Free tier|—|$0.00|
|**Total (baseline 20 meetings)**||||**\~$44 / month**|

\---

### Scaled Cost Table

|Volume|Transcription|AI tokens|Orchestration|**Total est.**|
|-|-|-|-|-|
|10 meetings/month|\~$2.70|\~$1.50|$20.00|**\~$24**|
|20 meetings/month (target)|\~$5.40|\~$3.00|$20.00|**\~$28–44**|
|40 meetings/month|\~$10.80|\~$6.00|$20.00|**\~$37–57**|
|100 meetings/month|\~$27.00|\~$15.00|$20.00|**\~$62–82**|

**This solution sits well within Sarah's $1,500/month ceiling at any realistic volume.**

\---

### Variable Cost Sensitivity

|Driver|Impact|
|-|-|
|Longer meetings (60 vs 30 min)|Doubles transcription cost (\~+$5/month at 20 meetings)|
|Switching to Claude Opus 4|\~5× output cost; \~$7.50 extra/month at baseline|
|Adding supporting doc analysis|+\~$1–2/month in token costs|
|Scaling to 100 meetings/month|Still under $100/month total|

\---

### One-Time Setup Costs (estimate)

* Developer/consultant time to build and test: 8–16 hours
* Google Drive webhook + Drive API setup: included in above
* Initial prompt engineering and QA: 4–8 hours
* **No software licence fees beyond above**

\---

\---

## 

