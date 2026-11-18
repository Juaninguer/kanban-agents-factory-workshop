# Lab journal

> Fill this in **as you go** (5 minutes at the end of each part), not at the end of the session.
> It is graded on honesty and insight, not on how well everything went.
> Failures, surprises and "the agent did something stupid" moments are the most valuable content.

**Pair:** Juan Manzano Rus (solo).
**Repository:** https://github.com/Juaninguer/kanban-agents-factory-workshop

---

## Part 0 — Setup

- Copilot plan and VS Code version: Copilot Free (my Student verification is pending), latest VS Code.
- AI credits remaining at the start of the session: full monthly Copilot Free allowance (not used yet).
- Anything that did not work out of the box: the repo had no "Use this template" button, so I forked it and enabled GitHub Actions by hand. My Student application was rejected (missing 2FA, billing name and a verified email). `git push` was also refused at first because Git was using another GitHub account's stored credentials; I fixed it by adding that account as a collaborator.
## Part 1 — First agent: the Planner

- Prompt(s) used to run the planner:
- What was good / bad in the first spec it produced?
- What did you change in `planner.agent.md` after the first run, and what effect did it have?
- Prompt(s) used to run the planner: `Plan F1.` (new session, planner agent selected), run twice.
- What was good / bad in the first spec it produced? Good: PATCH /api/tasks/:id justified, 6 Given/When/Then ACs including 400 and 404 errors, native <select> for keyboard use, only real file paths, no code written. Bad: it planned the e2e tests inside the existing e2e/board.spec.ts instead of a new per-feature file; it left the control's accessible name "implementation-specific", which the tester can't target with getByRole; its 5 "open questions" all had defaults, so they were really assumptions that would wrongly block the implementer.
- What did you change in `planner.agent.md` after the first run, and what effect did it have? Added a step forcing exact roles/accessible names, a self-check for a new e2e/<ID>-<slug>.spec.ts file and for how e2e state/failures are set up, a separate "Assumptions" section, and a constraint against modifying existing tests. Effect on the second run: e2e tests are now planned in a new e2e/F1-move-task.spec.ts, the control has an exact name (combobox `Move "<title>"`), AC5 explains how to force the failure with Playwright `page.route`, each e2e scenario states its initial board state, and the defaults moved to "Assumptions" with "Open questions: None". Downside: it went from 6 to 5 ACs (it merged "move to every column" into AC1/AC2).

## Part 2 — Context engineering

- What did you put in the project-wide instructions vs. path-specific instructions? Why?
- Did you observe a difference in the planner's output after adding the instructions?

## Part 3 — The factory, with handoffs

- Feature implemented (ID):
- For each agent (implementer, tester, reviewer): one thing it did well, one thing it did badly.
- What did the Playwright MCP server let the tester do that it couldn't do before?
- Did the tester load the `e2e-testing` skill? How did you check?
- Human interventions: how many times did you have to step in, and why?

## Part 4 — The orchestrator, with subagents

- Feature(s) implemented (IDs):
- Handoffs (Part 3) vs. orchestrator (Part 4): which gave the better result? Which one would you
  trust on a real project, and under which conditions?
- Did the orchestrator ever loop, skip a step or "lie" about a result? What did you change?

## Part 5 — Hooks & guardrails

- Which hooks did you add, and which event triggers each?
- Show one case where a hook changed the agent's behaviour (paste the relevant output).

## Part 6 — Copilot CLI (bonus)

- What did you run and how did it compare to VS Code?

## Part 7 — Final demo

- Feature request received:
- Number of human interventions needed:
- Final `npm run verify` result:

---

## Reflection (to fill at the end, ~10 lines)

1. Which part of the factory produced the most value? Which one the least?
2. Where would you **not** trust this factory today?
3. One thing you will change in the way you use AI coding tools after today.

## AI credits log

Copilot Student uses _auto model selection_ and a monthly allowance of GitHub AI Credits.
Note your usage at the start and at the end of each part (see Part 0 for where to find it).

| Part | Credits used (approx.) | Most expensive step, and why |
| ---- | ---------------------- | ---------------------------- |
|      |                        |                              |
