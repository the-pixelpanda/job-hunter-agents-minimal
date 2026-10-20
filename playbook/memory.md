# Memory & State — persistence, dedup, resume

Everything you know lives in `state/`. It is gitignored: never staged, never committed, never pushed, never copied into `playbook/` or any tracked file. If conversation context is lost, `state/` is the only survivor — write accordingly.

## State tree (create lazily, on first need)

```
state/
  progress.md              # session checkpoint: active modes, current step, last action, resume notes
  log.md                   # append-only activity log: "YYYY-MM-DD HH:MM | action | detail"
  decisions.md             # durable user rules, corrections, standing instructions (never overwritten)
  assumptions.md           # open assumptions awaiting user confirmation (cleared when resolved)
  candidate/
    profile.md             # identity, contact details, links, work authorization, notice period, location
    cv/                    # original CV files + cv-readable extracted text (pdf/cv.md/html as given)
    experience.md          # roles, employers, dates, responsibilities, achievements with metrics
    projects.md            # projects, portfolio, links
    skills.md              # hard skills, tools, transferable skills, skill→evidence map
    preferences.md         # preferences, dealbreakers, salary expectations, remote/on-site, industries
    criteria.md            # inclusion/exclusion criteria for a job being worth applying to
    target-jobs.md         # target titles, seniorities, search queries, geo, salary band
    qa.md                  # every discovery question asked and the user's answer (dedup Q&A here)
  accounts/
    index.md               # platform → accounts: email used, status (existing/created/verified), auth mode, credential reference
    <platform>.md          # per-platform playbook: apply flow, form-field map, quirks, workarounds, selectors, lesson learned
  answers/
    forms.md               # reusable application answers (salary, notice period, visa, refs, EEO, "why us"…) keyed by question
  jobs/
    ledger.jsonl           # dedup ledger: every job ever seen, one JSON object per line (schema below)
    queue.md               # human-readable view of pending jobs (derived from ledger; ledger is the truth)
  applications/
    ledger.jsonl           # every application ever submitted (schema below)
  reports/
    <YYYY-MM-DD>-apply.md  # end-of-run reports shown to the user
```

## Write rules

1. **Persist before replying.** Learned a fact → write it to the right file immediately, then talk.
2. **Append, don't rewrite.** `decisions.md`, `log.md`, `qa.md`, ledgers: append-only. Correction of a past item = append a new line noting the correction, or edit that one item in place — never bulk-rewrite.
3. **Checkpoint after every unit.** Update `progress.md` after each job batch, each application, each user answer. `progress.md` must always answer: what mode, what step, what's next, what was the last completed action.
4. **Read before ask.** Before asking the user anything, check `qa.md`, `answers/forms.md`, `decisions.md`, `profile.md`. Never ask what is recorded.
5. **One fact, one home.** A piece of knowledge goes in its designated file; other places reference it. Don't scatter duplicates.

## Deduplication algorithm (applies to search AND apply, everywhere)

Compute a job's canonical id in priority order:

1. **`platform:external-id`** — platform slug + the id in the job URL (path segment, numeric id, req number). Best key.
2. **Canonical URL** — lowercase host, strip scheme variance, strip query params/fragments/tracking (`?ref=`, `?utm_`, etc.), drop trailing slash → use as `id` if no external id.
3. **Fuzzy key** — normalize (lowercase, strip punctuation/diacritics, collapse whitespace) `title|company|location` → `id` if neither of the above is stable.

Rules:

- **Storing**: before appending to `jobs/ledger.jsonl`, grep the ledger for the id (and, for fuzzy keys, the title+company pair). Exists → update `last_seen` only, never append a second row.
- **Applying**: before touching a job, grep `applications/ledger.jsonl` for its id. Any row with `status` in `sent|interview|rejected|hired|withdrawn` → skip permanently, count as dedup hit. Also skip `status: blacklisted` in the job ledger.
- **Cross-platform**: the same opening posted on two platforms = two rows (different platform ids), but check title+company fuzzy key and warn the user before applying twice.

### `jobs/ledger.jsonl` row

```json
{"id":"linkedin:12345","title":"","company":"","location":"","url":"","platform":"","type":"full-time","salary":"","first_seen":"YYYY-MM-DD","last_seen":"YYYY-MM-DD","query":"","status":"new|queued|applied|expired|blacklisted","notes":""}
```

### `applications/ledger.jsonl` row

```json
{"job_id":"","date":"YYYY-MM-DD","platform":"","account":"","email_used":"","apply_mode":"autopilot|copilot","resume_used":"state/candidate/cv/...","status":"sent|interview|rejected|withdrawn","confirm_ref":"screenshot/email/ref no.","assumptions":["..."],"notes":""}
```

## Resume & recovery

- **Normal**: session start → read `progress.md` → announce the last action → continue.
- **Crash mid-run**: `progress.md` + per-unit checkpoints mean at most one unit is lost. Re-verify the last unit (check ledgers — if the application row exists, it was submitted; if not, redo it).
- **`state/` damaged/missing**: rebuild from `log.md`, the two ledgers, and `qa.md`; tell the user what could not be recovered and what needs re-entering.
- **Ambiguity**: when unsure whether something completed, check the ledgers first — a row's presence is the source of truth, not memory of the conversation.

## Learning loop (getting better over time)

- New platform insight (form layout, field meaning, workaround, selector, quirks) → append to `accounts/<platform>.md` immediately after use.
- New reusable form answer → `answers/forms.md`.
- User correction → `decisions.md`; confirmed assumption → move it from `assumptions.md` into the proper file and delete it from `assumptions.md`.
- Search that produced good hits → record the query in `target-jobs.md` as a proven query; dead queries → mark `# dead` next to them.
- Before each apply run, skim `accounts/<platform>.md` for the target platform — reuse past lessons.
