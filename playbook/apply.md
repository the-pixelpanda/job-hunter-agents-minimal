# Apply (Advance / Campaign) — browser-automation applications

Runs in **Advance** (apply to the saved queue only, no searching) or **Campaign** (search, then apply). Applies jobs from `state/jobs/queue.md` / `jobs/ledger.jsonl` via the harness's browser automation tools.

## Apply modes

| Mode | Key | Behavior |
|---|---|---|
| **Autopilot** | `autopilot` (default) | Never asks mid-run. Fills forms from state; where data is missing, makes the most reasonable assumption, logs it to `state/assumptions.md`, continues. Reports all assumptions at the end. |
| **Copilot** | `copilot` (default for the **first** apply session) | Works autonomously on everything known; asks the user only when genuinely stuck (unknown field with no reasonable default, captcha, payment, destructive/irreversible step, account creation needing phone/email validation not available). Every answer is recorded immediately and reused forever. |

User can switch mode at any time ("autopilot from now on" / "ask me when unsure"). Persist in `progress.md`.

## Setup (once per session)

1. Read `progress.md`, `decisions.md`, `accounts/index.md`, `answers/forms.md`, and `accounts/<platform>.md` for each platform in the queue.
2. Build the worklist: queue jobs, **minus** every id already present in `applications/ledger.jsonl`, minus `blacklisted`/`expired`.
3. If worklist is empty → report "nothing to apply to" (and why: all applied / queue empty) and stop.
4. Accounts: for each platform needed, check `accounts/index.md`:
   - Existing/pre-authenticated session → use it.
   - Credentials stored → log in with them.
   - No account → create one. Email: use a user-provided address/inbox (if inbox access exists, validate the verification email) or an address the user designates. **Never invent a personal identity** — invented identity details go to `assumptions.md`. Ask first in Copilot; assume in Autopilot.
   - Record every created account/credential in `accounts/index.md` immediately.

## Per-job loop (checkpoint after each job)

1. **Dedup re-check** — grep `applications/ledger.jsonl` for the job id; applied before → skip, count as dedup.
2. Open the posting; verify it is live. Dead → mark `expired` in job ledger, next.
3. Fill the application using, in order of authority:
   1. `answers/forms.md` (exact remembered answers)
   2. `profile.md` / `experience.md` / `skills.md` / `preferences.md`
   3. the CV in `candidate/cv/`
   4. `decisions.md` standing rules (e.g. "never state a notice period over 4 weeks")
   5. otherwise → **Autopilot**: best-effort assumption + log to `assumptions.md`; **Copilot**: ask the user, then append the answer to `answers/forms.md` (or `qa.md` if one-off).
4. Attach the CV; pick the right CV version if several exist (default: the latest; note the choice).
5. Review answers on the final page. Submit only applications — never confirm irreversible side effects (payments, agreements to terms the user hasn't sanctioned) without user go-ahead.
6. Capture proof: confirmation text/reference/screenshot path → append a row to `applications/ledger.jsonl`; update the job row `status: applied`.
7. Learned something about the platform (field meaning, workaround, selector, validation quirk, flow change) → append to `accounts/<platform>.md` right away.
8. Checkpoint `progress.md` (last applied job id, position in worklist).

## Interruptions & errors

- Session dies / browser breaks → on resume, read `progress.md` + `applications/ledger.jsonl`: rows present = done, absent = redo. Never resubmit a job that has a row.
- Form rejects input → log the error next to the job row, apply the fix, note the workaround in the platform playbook, retry once; second failure → mark `notes: blocked`, move on (Copilot may ask).
- Never lose track of position: `progress.md` always holds the worklist pointer.

## Finish — report (always, Autopilot especially)

Write `state/reports/<YYYY-MM-DD>-apply.md`, then show it in chat:

```
apply report — YYYY-MM-DD   mode: autopilot
submitted: N     dedup-skipped: D     failed/blocked: F     accounts created: A
per job: <title @ company> — sent [ref] / blocked [reason]
assumptions needing your review (numbered):
  1. <what was assumed, for which job/field>   → say "agree" or correct it
next: I'll wait — say "apply" to continue the queue.
```

Then handle the user's response:

- **Agreed** → move the assumption from `assumptions.md` into its permanent home (`answers/forms.md`, `profile.md`, or `decisions.md`) and remove it from `assumptions.md`.
- **Corrected** → apply the correction everywhere it was used (ledger notes, answers), write the rule to `decisions.md`, so it never happens again.

## Rules

- **Zero duplicate applications.** The `applications/ledger.jsonl` check is mandatory before every submission.
- Every submission, skip, account, and assumption is persisted before moving on.
- Copilot asks sparingly and batches questions; Autopilot never asks mid-run and never stops until the worklist is done or blocked by something only the user can resolve.
- The user may stop, redirect, or veto at any moment — immediately checkpoint and comply.
