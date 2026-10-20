# Search (Scout / Campaign) — finding jobs without duplicates

Runs in **Scout** (search only) or **Campaign** (search + apply). Never applies in Scout mode.

## 1. Build queries

- Source of truth: `state/candidate/target-jobs.md` (titles, seniority, geo, salary, proven queries) + `criteria.md`.
- No target jobs recorded → run discovery first (or the minimum: ask for target titles + location before searching).
- Compose a query set: title variants × location/remote × seniority; prefer queries previously marked proven; mark dead queries.

## 2. Search with whatever tools exist

Use the harness's search/browser tools across job boards and aggregators (LinkedIn, Indeed, Glassdoor, Otta/Welcome to the Jungle, Wellfound, Xing, Greenhouse/Lever/Ashby/Workday portals, company career pages, Google/ Bing site queries, etc.). Platform list is not fixed — use what the tools allow.

Tips:

- Prefer board-native filters (date posted, remote, seniority) to keep result quality high.
- Record the source query for every hit (it goes in the ledger row `query`).

## 3. Harvest loop (per result)

1. Extract: title, company, location, URL, platform, type, salary (if shown).
2. Compute the canonical id per `playbook/memory.md` dedup algorithm.
3. Grep `state/jobs/ledger.jsonl` for that id (and fuzzy title|company|location fallback).
   - Known → update `last_seen` only. Count as duplicate.
   - New → append a full row (`first_seen`, `status: new`).
4. Apply quick criteria from `criteria.md`: obviously-off-target hits → store anyway with `status: blacklisted` + reason (so they are never surfaced or applied later) if the mismatch is clear; otherwise keep `new`.

Batch-size rule: checkpoint `progress.md` every ~10 stored jobs so an interruption loses nothing.

## 4. Close out the run

1. Rebuild `state/jobs/queue.md` from the ledger (pending = status `new` or `queued`, not blacklisted/expired) grouped by date — this is the human view; the JSONL is the truth.
2. Append a `log.md` line: queries used, hits seen, new stored, dupes skipped.
3. Report to the user in one short block:

```
search report — YYYY-MM-DD
new: N   duplicates: D   queue total: Q
top matches: <3–5 title @ company>
next: say "apply" (advance) or "campaign" to search+apply
```

## Rules

- Never store the same job twice (dedup algorithm is mandatory, every result, every run).
- Never apply in Scout mode.
- Re-running the same queries must produce 0 duplicates — if it produces any, the dedup step was skipped; fix it before continuing.
- All state writes happen before the report is spoken.
