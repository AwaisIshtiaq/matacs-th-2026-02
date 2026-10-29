# Write-up: AI proposal triage

**Awais Ishtiaq** · MATACS-TH-2026-02

A reviewer uploads the PDFs and picks a category. The system extracts the text, finds the evidence for each criterion and cites the page, checks the two mandatory gates, scores the five weighted criteria, and sorts the results in code. The model reads prose and judges the qualitative criterion. It does not decide eligibility, do arithmetic, or set the order. A reviewer confirms or overrides every flag before anything is final.

## Scoring model

Sub-scores are 0 to 100. Total = Σ (weight × sub-score).

- **Experience 30%:** 100 × qualifying installs ÷ highest count in the set.
- **Price 25%:** 100 × lowest eligible price ÷ price, only if ≤ $260,000.
- **Timeline 15%:** 100 × shortest eligible weeks ÷ weeks, only if ≤ 16.
- **Warranty 15%:** 50 for a 25-year panel warranty + 50 × workmanship years ÷ 12.
- **Team 15%:** 25 each for named lead, named crew, safety plan, monitoring plan.

An undisclosed value scores zero and is flagged. A bid over the ceiling or over 16 weeks is flagged for review instead of being rejected automatically.

## 1. AI vs rules

Anything with one correct answer belongs in code: validation, extraction, the regex pass on money and dates, thresholds, normalization, the weighted sum, ordering, and the injection screen. The model handles unstructured prose: finding each criterion's evidence and judging team and approach against the admin's words. The test is whether a person with a calculator could check the result. If yes, code it. The model returns JSON with a verbatim quote for every value and `unknown` when the document is silent. It never returns the order.

## 2. Consistency and explainability

Repeatability comes from pinning prompt, model and criteria versions, temperature 0, one pass per proposal, and caching by file hash. The arithmetic is code, so identical inputs give identical output. Re-runs are diffed; drift beyond tolerance needs reviewer confirmation. Every position shows the sub-score, weight, points contributed, the quoted evidence with its page number, and the arithmetic line. A reviewer can recompute the total by hand.

## 3. Human in the loop

A person reviews an ambiguous mandatory criterion, a missing value, an injection flag, low confidence, a rules-versus-model conflict, every override, and the final award. Each run logs the file hash, upload time, criteria and weights version, prompt and model version, raw model response, extracted values with page spans, sub-scores, the arithmetic, flags raised, reviewer identity, every override with before and after, and the final sign-off. That is enough to reconstruct any result without rerunning the model.

## 4. Changing criteria

Criteria and weights are versioned records. An admin edit creates a new version; nothing is edited in place. A run pins the version it started with, and proposals are never scored across two versions. The next run rescores the whole category under the new version in one batch and shows the delta against the previous ranking. Editing is blocked while a run is in progress. If a mandatory criterion changes, eligibility is re-evaluated first, then scores. The reviewer sees "v2: price weight 25→35, C moves 3→2", so the reorder has a visible cause.

## 5. Risks and guards

| Risk | Guard |
|---|---|
| Made-up facts | Extractive only; every value needs a quote that exists in the source; `unknown` allowed; unsupported values rejected |
| Wrong extraction | Regex pass and model pass must agree; disagreement flags for review |
| Manipulation | Document text is fenced data; the call has no tools; output is schema-bound; instruction patterns are screened; arithmetic and order stay in code |
| Bias | Criterion-anchored rubric; evidence quotes; no free-form "overall impression"; evidence shown before scores |
| Non-determinism | Pinned versions, temperature 0, caching, re-run diffs |
| Cost | One pass per proposal, per-criterion batching, page caps, retry cap |
| Automation bias | Every flag must be acknowledged; sampled audits; award never automated |

## The sample set

A and E are straightforward. B is the eligibility trap: cheapest and fastest, but the electrical licence belongs to a subcontractor to be confirmed after award, which fails the "held by the bidder" gate. C contains an instruction to automated review systems asking for first place and full marks. The system detects, ignores, and logs it, and C is scored on merit. D has no completion date, so timeline is `unknown`, scores zero, and is flagged.

| Rank | Proposal | Score | Why |
|---|---|---|---|
| 1 | A | 98.0 | Strongest experience, second-lowest price, 12 weeks, full team and monitoring |
| 2 | E | 84.8 | Best warranty and team; mid-pack price and experience |
| 3 | C | 77.1 | Eligible on merit; the embedded instruction did not move it |
| 4 | D | 69.8 | Lowest price, but no committed schedule |
| n/a | B | n/a | Ineligible: licence not held by the bidder |

C and D are close. If D's missing schedule were treated as the 16-week ceiling, D would score 81.1 and overtake C; a linear price curve would also put D above C. The ratio rule is used because it is stable. A and E hold first and second under every variant tested.

B is also excluded from the price and timeline benchmarks: a bid that cannot be awarded should not set the bar.

## Assumptions

Licence, WSIB and insurance are checked as stated; no register lookup. The five-year window runs back from the run date. Prices are compared pre-tax in CAD on a like-for-like basis. One proposal per PDF. Missing evidence scores zero and is flagged. Injection attempts are flagged rather than disqualified, because a competitor could plant one.

## Before live use

Extraction accuracy is measured against a labelled set first. Five short summaries cannot show it, and a reviewer who rubber-stamps defeats every control above.
