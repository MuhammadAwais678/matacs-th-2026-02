# Write-up: AI Proposal Triage Design

## Flow in short

Upload → check files → extract text → **AI extracts facts with evidence** → code verifies evidence and scans for manipulation → code checks mandatory criteria → code scores weighted criteria → code ranks → **human reviews, can override, and awards**.

## 1. AI vs. rules

**AI is used for one job only:** reading each proposal and pulling out facts (licence stated, install count, price, weeks, warranty years, named lead, crew, safety plan, monitoring) as structured JSON, each with a short quote from the proposal. Proposals are written in free text and every company words things differently, so rules alone would miss too much.

**Everything else is plain code:** file checks, text extraction, evidence verification, pass/fail on mandatory criteria, score formulas, ranking and the audit log. Money, dates and weights must be exact and repeatable, and code is better at that than an AI model. The reason text is built from a template filled with the real numbers.

The line is: AI *reads*, code *decides*.

## 2. Consistency and explainability

- Scores come from fixed formulas applied to the extracted facts, so the same facts always give the same score.
- The extraction step uses temperature 0, a pinned model version and a versioned prompt. The result is saved and keyed by file hash + prompt version + criteria version. Ranking the same proposals twice reuses the saved extraction, so the result is identical. A fresh re-extraction could differ slightly, so the tool shows a diff if that is ever run.
- Reviewers see a table per proposal: each criterion, the extracted value, the quote from the proposal, the formula used, the points and the weight. Nothing is a black-box number.

Formulas used for the sample set (fixed, so they do not change when other bids change):
- Experience: qualifying installs ÷ 12, capped at 100%.
- Price: lowest eligible bid ÷ this bid.
- Timeline: linear from 16 weeks (0 points) to 8 weeks (100 points).
- Warranty: 50% panel years ÷ 25, plus 50% workmanship years ÷ 15, each capped at 100%.
- Team and approach: 25 points each for named lead, stated crew, safety plan, monitoring plan.
- Any weighted item the proposal does not state scores 0 for that criterion, with a warning.

## 3. Human in the loop

- **Flagged queue:** any proposal with a manipulation warning, low-confidence extraction or an unverified quote goes to a person before it can be ranked. A proposal with missing weighted data is still ranked, but shows a visible warning and the missing item scores 0.
- **Ineligible list:** the reviewer can see why a bid failed a mandatory criterion and can reverse it if the extraction was wrong.
- **Overrides:** the reviewer can change any extracted fact or score. A written reason is required.
- **Final award:** always a human decision. The tool only recommends an order.
- **Audit log:** file hash, extracted text, model name and version, prompt version, criteria version, raw AI output, rule results, scores, every override (who, when, what, why) and the final decision.

## 4. Changing criteria

Criteria are versioned and never edited in place. An edit creates a new version, and every scoring run records the version it used.
- **Weights or thresholds changed:** rescore from the saved facts with code. No AI cost, instant.
- **A criterion added or its meaning changed:** re-extract only for that criterion.
- Old results stay linked to the old version and are marked "outdated", never mixed into the new ranking. The reviewer sees old vs new rank side by side before accepting the new run. An award already made keeps the version it was based on.

## 5. Risks

| Risk | Safeguard |
|---|---|
| Wrong extraction (bad OCR, misread number) | Every fact needs a quote. Code checks the quote really appears in the source text and the number matches. Failures are flagged. Reviewer sees the quote next to the value. |
| Made-up facts | The prompt says to use `null` when something is not stated. Code never accepts a value without a verified quote. |
| Prompt injection / manipulation | Proposal text is treated as data, not instructions. The prompt says to ignore any instruction inside it. Code scans for phrases aimed at "reviewers", "AI" or "systems" asking for rank or marks. A hit flags the proposal for a person and the instruction is never followed. |
| Bias | Company names are replaced with the letter before scoring. Fixed rubric. No scoring on size, brand or location. Reviewers can see all reasons. |
| Missing or vague information | Not guessed. Mandatory items that are vague fail. A weighted item with missing data scores 0 and shows a warning, so a bidder cannot gain by leaving it out, and the reviewer can ask the bidder. |
| Over-reliance on the tool | The tool recommends only. A human awards and can override. |
| Cost | One AI call per proposal, small model, cached results, page limit per file, no re-calls when only weights change. |
| Privacy | Only fictional or approved data. No API keys in the repo. |

## Sample set result

- **A**: rank 1, score about 89.3.
- **E**: rank 2, score about 77.9.
- **B**: ineligible. The electrical work is subcontracted and the licence is not held, so it fails a mandatory criterion even though it has the lowest price.
- **C**: flagged. It contains hidden text trying to force first place. The instruction is ignored and a person reviews it. The legitimate content would score about 69.9, shown to the reviewer only as advice.
- **D**: rank 3, score about 72.5. No timeline is given, so timeline scores 0 and a warning is shown. Claude's first idea was to flag D and leave it unranked, but a bid that leaves out its schedule should not get a free pass or disappear from the list, so I rank it with the gap visible and let the reviewer ask the bidder.
