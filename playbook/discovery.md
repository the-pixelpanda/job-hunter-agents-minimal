# Discovery — getting to know the candidate

Asked at session start on first run (or whenever the user says "discover me again" / "update my profile"). Pick one:

| Mode | Key | Time | What happens |
|---|---|---|---|
| **Quick Intake** | `quick` | ~10 min | Minimal viable profile |
| **Standard Intake** | `standard` | ~15 min | Solid working profile |
| **Deep Discovery** | `deep` | iterative | Full mapping, loops until converged |

The user picks; record the choice in `state/progress.md`.

## Before asking anything

Read `state/candidate/` — if a discovery ran before, you are **refreshing**, not starting over: say what you already know, ask only about gaps and changes.

## CV intake (all modes, first step)

1. Ask for the CV: original file (PDF preferred) **and** a readable format (plain text, Markdown, or HTML). If only a file is given and you can extract text, extract it; if you cannot, ask for the readable version.
2. Save originals + extracted text to `state/candidate/cv/`.
3. Parse into `profile.md`, `experience.md`, `projects.md`, `skills.md` — then confirm your parsing back to the user briefly ("here's what I extracted") before continuing.

## Quick Intake (`quick`) — batch these in 1–2 messages

1. CV (file + readable) — required.
2. Target roles / titles you want (and the ones you never want).
3. Seniority level sought.
4. Location, remote preference, work authorization, notice period.
5. Salary expectation (number or range, currency).
6. Anything else the user volunteers.

Write answers to their homes (`profile.md`, `target-jobs.md`, `preferences.md`, `qa.md`). Done → propose execution modes.

## Standard Intake (`standard`) — everything in quick, plus:

- Experience highlights: last 2–3 roles, biggest achievements with numbers.
- Skills: hard skills, tools/stack, **transferable skills**.
- Industries and company sizes that fit; industries to avoid.
- Dealbreakers (travel %, on-site days, on-call, sector, etc.).
- Platforms/accounts already held (LinkedIn, Indeed, company portals…) → seed `state/accounts/index.md` (ask whether credentials can be stored locally in `state/accounts/`).
- Availability details: start date, references, visa/sponsorship needs, gaps in CV worth pre-explaining.
- Anything on application forms they always get asked and hate → start `answers/forms.md`.

## Deep Discovery (`deep`) — loop until converged

Cover everything in `standard`, then iterate in rounds:

- Round questions on: projects & portfolio, metrics/impact per role, certifications & education, talks/writing, leadership scope, tooling depth per skill, target-company list, cover-letter voice/preferences, typical form questions (EEO, salary, notice period, referrals, relocation, "why us" templates), account inventory with verification status, email inbox access for validations.
- After each round, summarize the profile back; user corrects; apply corrections (user override rule).
- **Convergence test**: stop when a full round produces no new material facts and the user agrees the profile is complete. Record `discovery: deep — converged YYYY-MM-DD` in `progress.md`.

## Every answer, every time

1. Append the Q&A to `state/candidate/qa.md`.
2. Route the fact to its file (`profile.md` / `skills.md` / `preferences.md` / …).
3. Checkpoint `progress.md`.

## After discovery

Offer the execution mode menu (`scout` / `advance` / `campaign`) and the apply mode (`copilot` first time, `autopilot` after). Persist choices to `progress.md`.
