# Job Hunter

A **codeless agent framework** for searching and applying to jobs. No scripts, no dependencies — just instructions an AI agent (any harness) follows, plus a local, gitignored knowledge base that grows over time.

## How it works

- **Agent instructions** (committed, tiny): `AGENTS.md` is the entry point; `playbook/` holds the workflows. The agent reads a playbook file only when the task calls for it.
- **Knowledge base** (`state/`, gitignored, local-only): everything the agent learns — your CV, skills, preferences, rules, answers, platforms, accounts, jobs seen, applications sent, assumptions, platform playbooks, reports. Never committed, never pushed.
- **Native deduplication**: every job has a canonical id (`platform:id` → canonical URL → normalized title|company|location). Jobs are stored once; applied jobs are never applied to twice.
- **Crash-proof**: after every unit of work the agent checkpoints `state/progress.md`. New session → it resumes where it left off.

## Quick start

1. Clone this repo into an empty harness workspace (or copy it in).
2. Open a chat. The agent reads `AGENTS.md` (via `CLAUDE.md` in Claude Code, or point your harness's instruction file at it).
3. Say **"I want to apply for jobs"**.
4. Pick a discover mode, drop your CV, answer its questions.
5. Pick an execution mode — **Scout** (search), **Advance** (apply saved list), **Campaign** (search + apply).
6. When applying: **Copilot** asks you when it's stuck (first session); **Autopilot** runs alone and reports assumptions at the end.

## Modes

| Dimension | Modes |
|---|---|
| Discover | Quick Intake (~10m) · Standard Intake (~15m) · Deep Discovery (loop until converged) |
| Execution | Scout (search only) · Advance (apply saved list) · Campaign (search + apply) |
| Run | Candidate (default) · Owner (explicit order — edits the framework) |
| Apply | Autopilot (default, never asks mid-run) · Copilot (asks when stuck; first-session default) |

## Structure

```
AGENTS.md            entry point: directives, mode menu, routing, hard rules
CLAUDE.md            one-line pointer for harnesses that read their own file
playbook/
  memory.md          state schema, dedup algorithm, resume/recovery, learning loop
  discovery.md       discover modes & question flows
  search.md          Scout/Campaign search workflow
  apply.md           Advance/Campaign apply workflow, accounts, reports
  owner.md           Owner mode: how the framework edits itself
state/               runtime knowledge — gitignored, created by the agent
```

## Guarantees

- **You are the owner.** The agent obeys you; you own the risk. It warns once on clearly dangerous/illegal asks, then complies if you insist.
- **No code, ever.** The agent uses only the harness's built-in tools (search, files, chat, browser automation).
- **`state/` never leaves your machine** (`.gitignore` + agent hard rule). No pushes, no commits of state.
- **Harness-agnostic**: instructions live in plain markdown; clone into any harness that supports agent instructions and it works immediately.

## Changing the framework

Say **"enter owner mode"** — the agent proposes changes, waits for your approval, edits only tracked files, and never touches `state/` unless you order it.
