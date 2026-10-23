---
name: ship
description: Take finished local work to a reviewed PR — stop this worktree's dev servers, branch, commit, open the PR, babysit it to clean, and merge only when the user granted it. Use when the user says "open a PR and babysit", "ship it", or "kill the dev servers and open a PR".
---

# Ship

Local work → clean PR, in one pass. The user should only have to say it once.

## 1. Stop this worktree's dev servers

Follow the **Stopping** section of the `show-me` skill. Skip only if the user said to keep them up. The completion criterion: no process with a working directory inside this worktree is still listening on a port.

## 2. Branch, verify, commit, push

- Already on a feature branch → stay on it. On `main`/`develop` → cut `feature/<name>` (or `bugfix/<name>`) from `develop` if it exists, else `main`.
- Run the repo's full verification gate (`pnpm verify`, or whatever the repo's scripts and agent docs define). It must pass before pushing.
- Commit in Conventional Commits form with the model `Co-Authored-By:` trailer, then push.

## 3. Open the PR

- `gh pr create`, ready for review (draft only if asked). The body explains what changed and why for a reviewer, then ends with the model-provenance sentence.
- In T3 Code, call `link_pull_request` with the PR URL.

## 4. Babysit to clean

Follow the `babysit-pr` skill for triage, fixes, thread replies, and delta re-reviews. In T3 Code, wait with `watch_pull_request` and end the turn instead of running a polling loop; each wake-up is one triage round.

The PR is **clean** only when all three hold, checked against a fresh fetch of the PR at its current HEAD:

- CI is green with zero pending checks.
- The latest bot review covers HEAD and has no unaddressed findings.
- Every review thread and issue comment has a reply from you (fixed in `<sha>`, or disputed with evidence).

Re-fetch every comment before saying "clean". A summary from an earlier round is stale.

## 5. Merge gate

Merging is the user's call, made per PR. Merge only when the user explicitly granted it in this thread, e.g. "merge it", "if you get the thumbs up, merge", "merge when it passes". This overrides `babysit-pr`'s never-merge rule for that PR only.

- **Granted** → once the PR is clean, merge with `gh pr merge <n>` using the repo's allowed method (prefer `--squash`; check `gh repo view --json squashMergeAllowed,mergeCommitAllowed`). Confirm the state with `gh pr view <n> --json state`.
- **Not granted** → stop at clean and report it ready. A later "merge it" is the grant.
- **A finding needs a product or risk decision** → stop and ask, even when merging was granted.

## 6. Hand back

In T3 Code, call `unwatch_pull_request` first. Then report in this order: merged / ready / blocked, the PR link, CI status, and a short list of findings: fixed (with commits), disputed (and why), left for the user.
