# AI Proposal Triage Design — MATACS Associate Software Developer Assessment

* **Candidate:** Michael Gibb
* **Reference:** MATACS-TH-2026-02

* **Time Spent:** 1 hour 30 minutes

## Repository Contents
* [Process Flow Diagram (`diagram/process-flow.png`)](./diagram/process-flow.png) — End-to-end architecture showing AI extraction steps, deterministic rule gates, and human-in-the-loop checkpoints.
* [Design Write-Up (`WRITEUP.md`)](./WRITEUP.md) — Answers to the five core system design, consistency, governance, and risk questions.
* [AI Prompts & Schema (`PROMPTS.md`)](./PROMPTS.md) — System and user prompt templates, runtime dynamic criteria injection format, and strict JSON extraction schema.
* [Sample Evaluation Output (`ranking.json`)](./ranking.json) — Final ranked, flagged, and ineligible outputs for sample proposals A–E.

## AI Tool Usage & Corrections
I used an AI assistant to format the flow diagram, proofread my write-up, and turn my plain-text prompt outline into the structured Markdown and JSON schema in `PROMPTS.md` through back-and-forth iteration. When discussing criteria updates and cost risks, the AI initially suggested re-running the entire batch of proposals whenever criteria change and used overly complex cryptographic caching jargon to explain cost control. I corrected both points in `WRITEUP.md` by adding a practical rule to skip re-running any `ineligible` proposal under `v2` if the specific mandatory criterion it failed did not change, and by simplifying the cost safeguards into plain language that is easy for non-developers to follow.
