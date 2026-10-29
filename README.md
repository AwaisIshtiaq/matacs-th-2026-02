# README: MATACS-TH-2026-02

**Awais Ishtiaq** · Reference **MATACS-TH-2026-02**

Time actually spent: **about 3 hours** (requirements analysis, scoring model, prompts, write-up, diagram).

## What this is

A design for an AI-assisted tool that triages contractor proposals. An admin defines the criteria, a reviewer uploads PDFs, and the system extracts evidence with page citations, checks the mandatory gates, scores the weighted criteria, and returns a ranked list with reasons. A person makes the final decision. No application is deployed.

## How I used AI

I used a general-purpose AI assistant (DeepSeek) to extract the brief's requirements, pressure-test the scoring model, and draft the prompts and write-up. I corrected it twice. It first read the PDF's scrambled table columns as Timeline 25% / Price 15%, when the plain-text brief shows **Price 25% / Timeline 15%**. It also put the supporting analysis files inside the submission folder, which conflicts with the brief's fixed layout, so I moved them out. I overruled its first instinct on the injected proposal too: I chose to flag rather than auto-disqualify, because disqualifying on a phrase is itself exploitable.

## Files

| File | What it contains |
|---|---|
| [WRITEUP.md](WRITEUP.md) | Answers to the five questions (under 1,000 words) |
| [PROMPTS.md](PROMPTS.md) | The extraction and scoring prompts, exactly as sent |
| [ranking.json](ranking.json) | Expected ranking of proposals A-E |
| [diagram/process-flow.png](diagram/process-flow.png) | Upload-to-ranking flow, with AI, rule and human steps |

The required files are at the paths above. The working notes behind them (requirements, edge cases, assumptions, product spec, architecture) are kept outside this folder so the repository matches the brief's layout.

## Summary of the result

A (98.0) → E (84.8) → C (77.1, flagged for an embedded instruction, which was ignored) → D (69.8, flagged for having no committed schedule). B is ineligible: the mandatory electrical licence is held by a subcontractor who is still to be confirmed, not by the bidder.

The ranking is reproducible by hand from the rules in `WRITEUP.md`. C and D are close, and their order depends on policy: whether a missing value scores 0 or the 16-week ceiling, and which price curve is used. Both choices are stated in `WRITEUP.md`.

## Notes

- The assessment brief is not reproduced in this repository, per its redistribution terms.
- Proposals are referred to by letter only.
- No real client or personal data is used.
