---
name: babysit-pr
description: Watch an open pull request for bot review findings and CI results, triage each finding, fix the valid ones, push, and request delta-scoped re-reviews until the PR is clean. Use when the user says "babysit", "watch the PR", "handle review feedback", or asks to fix and repush as bot findings come in.
---

# Babysit PR

Sit on an open PR after pushing: wait for bot reviewers (Codex, CodeRabbit, Copilot, etc.) and CI, judge each finding on its merits, fix what's valid, reply to every thread, and re-review only the delta. End when the PR is clean or a finding needs a human decision.

The loop is: **poll → triage → fix → reply → re-review → poll**, with the user able to interject at any point because all waiting happens in background tasks.

## 1. Establish the baseline

Resolve the PR (`gh pr view`, or ask for the number if the branch has none). Record counts before watching, so only *new* activity triggers triage:

- top-level review comments: `gh api repos/<owner>/<repo>/pulls/<n>/comments --jq '[.[] | select(.in_reply_to_id == null)] | length'` — replies don't count, or your own replies will retrigger the loop
- submitted reviews: `gh pr view <n> --json reviews --jq '.reviews | length'`
- issue comments (some bots post summaries there, not as reviews)

## 2. Poll in the background

Run the watch loop as a **background** Bash task (never foreground sleep) so the session stays interactive and its exit re-invokes you:

- Check every ~45s, cap the task near 30 minutes; if it times out with no activity, do one final check and report rather than looping forever.
- Exit the loop and print a distinct marker for each condition:
  - `NEW_FINDINGS` — top-level review comments or reviews above baseline
  - `CI_FAILING` — any failing check (CI failures are findings too)
  - `ALL_CLEAR` — bot review finished, no new findings, zero pending checks; require a few successful polls before trusting it (bots take ~1–5 min to even start)
- Detect an in-flight bot review where possible (e.g. Codex's summary comment says "Running") and don't declare ALL_CLEAR while one is running.
- After the task fires, read its output file for the marker, then fetch the actual finding bodies with `gh api .../pulls/<n>/comments`.

## 3. Triage — bots are not automatically right

Judge each finding against the repository's own standards (its agent instructions, contributing docs, safety policies) and the actual code:

- **Valid** → fix it. Prefer the minimal change that genuinely resolves the issue; a finding about a claim you made (in docs or comments) can be fixed by correcting the claim *or* the code — pick whichever is actually right.
- **Invalid or out of scope** → do not change code to appease the bot. Reply on the thread explaining why, with evidence (file/line, doc reference), and move on.
- **Needs a product or risk decision** → stop babysitting and surface it to the user; don't guess on their behalf.

## 4. Fix, verify, push

- Apply fixes as focused commits following the repo's commit conventions.
- Run the repository's full verification gate (tests, lint, build — whatever the repo defines) before every push. Never push a fix you haven't verified locally.
- Push to the PR branch. Do not force-push during a review cycle; reviewers and threads anchor to commits.

## 5. Reply to every finding, then re-review the delta only

- Reply on each finding's thread naming the commit that addressed it:
  `gh api repos/<owner>/<repo>/pulls/<n>/comments/<comment_id>/replies -f body="Addressed in <sha> — <one line>."`
  Disputed findings get the evidence reply from step 3 instead. Never leave a finding thread unanswered, and never resolve threads yourself — that's the reviewer's call.
- Request a re-review **scoped to the new commit only, never the full PR** — full re-reviews re-flag old code and create an endless loop. If the `gh-rereview-latest-commit` skill is available, use it; otherwise post a PR comment naming the full SHA and stating that earlier commits were already reviewed.

## 6. Repeat, with a cap

Go back to step 2 with fresh baselines. Bound the process:

- After **3–4 rounds** without reaching clean, stop and report — recurring rounds usually mean the bot is nitpicking or a disagreement needs the user.
- Wind down when: the bot's latest pass is clean **and** CI is green (report the PR as ready), or all remaining findings are disputed-with-reply, or a finding is blocked on the user.

## 7. Final report

State the outcome first: PR ready or blocked, latest review verdict, CI status. Then the ledger: findings received, which were fixed (with commits), which were disputed and why, and anything left for the user. Report faithfully — a skipped or disputed finding is stated as such, not glossed as fixed.

## Safety

- Never merge, close, or mark threads resolved; babysitting ends at "ready for human review."
- Only reply and push to the PR's own branch; don't touch other branches or PRs.
- Quote all comment bodies safely; never interpolate finding text into shell unquoted.
- If the user pushed new commits themselves mid-loop, rebaseline instead of assuming your state is current.
