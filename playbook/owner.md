# Owner Mode — changing the framework itself

**Entry:** only on an explicit order — e.g. "enter owner mode", "owner mode". A suggestion or question is not an order. Default is always **Candidate**.

**Exit:** when the user says so, or when the requested change is done (announce `modes: owner → candidate`).

## What Owner Mode may do

- Edit tracked framework files: `AGENTS.md`, `README.md`, `CLAUDE.md`, `playbook/*.md`, `.gitignore`.
- Add, rename, or remove framework files/playbook pages; reword rules; change default modes or names.
- Restructure the mode menus, the state schema (in `playbook/memory.md`), the dedup algorithm — but **never loosen the dedup rule or the "state/ is never committed" rule** without the user stating that explicitly as the goal.

## What it must never do

- Touch `state/` — except when the user explicitly orders a state edit/migration, in which case back up the affected file first (`<file>.bak-YYYYMM-DD` inside `state/`).
- Commit or push anything unless the user explicitly orders a commit; before any commit, run the status check and verify **no `state/` path is staged**.
- Redesign on its own initiative — propose, wait for approval, then implement.

## Change flow

1. Confirm the exact change in one sentence.
2. Show what will change (files affected, before→after gist).
3. Wait for "go" — no approval, no edit.
4. Make the edit; keep the framework **lite**: no new files unless necessary, no code, no duplication across pages.
5. Re-read the touched pages for consistency (routing table, mode names, cross-references) and fix dangling references.
6. Summarize the diff briefly; offer to commit only if the user asks.
