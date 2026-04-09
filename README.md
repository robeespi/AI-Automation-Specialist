### Main goal

Build an AI automation workflow that uses the recording and transcript of a video meeting
to automatically generate a Slide deck after the meeting ends. The slide deck
should include an executive summary, 3 high-level objectives, 3 actionable items, and next
steps. Also,  include a separate document with estimated ongoing technology
costs associated with this solution (software licenses, cloud services, API fees, etc).
Considering the entire end-to-end journey, from ingesting raw media through to delivering
polished outputs.

### Part 1 — Solutions Architecture

- Core components required to take unstructured meeting data through
to a set of structured outputs such as ingestion, processing, AI inference,
formatting, storage, and delivery.
- solution architecture diagram. Including
integration points, data flow directions, and any triggers or schedulers you would
employ.
- Services, platforms, software, APIs/webhooks, cloud services and
coding languages specifications.

---

### Part 2 — Technology Cost Estimate

Estimation of the monthly recurring technology costs for this solution.

- Include platform licenses, API usage fees, storage or hosting, if applicable, and any
per-unit costs you anticipate (e.g., transcription minutes, model tokens).
- Inclyde different tiers of usage (at least 20 of these meetings per month) and highlight
any variable costs that scale with volume.


### Part 3 — The Functional Build (POC)

Proof of Concept (POC) of the primary workflow .

- minimal working version of the process using a jupyer notebook and Anthropic api. This poc
summarise the transcript into the requested elements, and output a presentation file. Focus on demonstrating the
flow rather than styling.
- A screen-share video is available that illustrate the steps taken and the
intermediate and final outputs generated. 

---

### Part 4 — Documentation (Dual-Audience)

1. For the Client (Non-Technical): Explanation of what the solution does, how it helps their
business, and how they would use it.

3. For the Team (Technical): Explanation of how the automation is structured, where the API
keys are managed, and how to troubleshoot it if it breaks. Including environment
setup, configuration notes, and guidance on extending or modifying the workflow.
Mention any logging and monitoring you would recommend.

