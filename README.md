# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Muhammad Awais
**Reference:** MATACS-TH-2026-02
**Role:** Associate Software Developer
**Time actually spent:** TODO: 60-minutes

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [diagram/process-flow.png](diagram/process-flow.png): flow from upload to ranked output (AI steps, rule steps, human checkpoints)
- [PROMPTS.md](PROMPTS.md): the extraction prompt, exactly as it would be sent
- [ranking.json](ranking.json): expected ranking of proposals A to E

## Summary of the design

AI reads messy proposal text and returns structured facts with an evidence quote for each. Plain code then checks the evidence, applies pass/fail rules, calculates scores with fixed formulas and ranks. A person reviews flagged items, can override any result with a reason, and makes the final award decision.

## How I used AI tools

TODO: edit this to match what really happened. Write 2 or 3 sentences. Say which AI tool you used (e.g. Claude) and for what (e.g. first draft of the flow and prompt). You must include at least one real place where you disagreed with or corrected the AI. Example shape: "Claude first suggested letting the AI model calculate the final scores. I changed this so scores come from fixed formulas in code, because the same proposal must always get the same score."

## Assumptions

- Experience is counted from what each proposal states, within the 5-year window where dates are given.
- A mandatory criterion passes only if the proposal states it clearly. "To be confirmed later" counts as a fail.
- A proposal with missing information on a weighted criterion is flagged for a human, not guessed.
