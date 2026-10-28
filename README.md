# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Muhammad Awais
**Reference:** MATACS-TH-2026-02
**Role:** Associate Software Developer
**Time actually spent:** TODO:20 minutes

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [diagram/process-flow.png](diagram/process-flow.png): flow from upload to ranked output (AI steps, rule steps, human checkpoints)
- [PROMPTS.md](PROMPTS.md): the extraction prompt, exactly as it would be sent
- [ranking.json](ranking.json): expected ranking of proposals A to E

## Summary of the design

AI reads messy proposal text and returns structured facts with an evidence quote for each. Plain code then checks the evidence, applies pass/fail rules, calculates scores with fixed formulas and ranks. A person reviews flagged items, can override any result with a reason, and makes the final award decision.

## How I used AI tools

I used Claude to help draft the process flow, the extraction prompt, the write-up, the diagram and the expected ranking, and I read through the outputs before using them. I disagreed with Claude on Proposal D: Claude flagged it and left it unranked because no timeline was given, but I changed it so D is ranked with a timeline score of 0 and a visible warning. A bid that leaves out its schedule should not get a free pass or disappear from the list, and the reviewer can still ask the bidder.

## Assumptions

- Experience is counted from what each proposal states, within the 5-year window where dates are given.
- A mandatory criterion passes only if the proposal states it clearly. "To be confirmed later" counts as a fail.
- A proposal with missing information on a weighted criterion scores 0 for that criterion and shows a warning, not a guess.
