---
name: gh-rereview-latest-commit
description: Trigger a GitHub Codex re-review limited to the current pull request's latest commit. Use when the user asks to retrigger Codex review, re-review recent findings or fixes, or review only the newest commit instead of the full PR.
---

# Re-review Latest PR Commit

Post a targeted `@codex review` request for the PR's remote head commit. Do not change code, push, resolve threads, or wait for the review unless the user separately asks.

## Workflow

1. Resolve the current repository and PR with local Git context and `gh pr view`.
2. Read the full remote `headRefOid`; treat it as the authoritative latest PR commit.
3. Compare it with `git rev-parse HEAD`. If they differ, mention the mismatch, but target the remote PR head unless the user asked about an unpushed local commit.
4. Derive a short focus from the user's request, such as "the recent review fixes." If no focus is given, use "the changes introduced by the latest commit."
5. Post one PR comment in this form:

   ```text
   @codex review

   Review only the latest commit, `<full-head-sha>`, relative to its first parent. Focus specifically on <focus>. Do not re-review earlier commits or the full pull request diff.
   ```

6. Return the PR comment URL and the short commit SHA. Do not claim the review completed; only report that it was triggered.

## Safety

- Use `gh` for GitHub operations.
- Post only after the user explicitly asks to trigger or retrigger the review.
- Preserve the full SHA in the comment so the scope cannot drift if another commit is pushed later.
- Pass the comment as a safely quoted multiline body; never evaluate user-provided focus text as shell code.
- If no PR is associated with the current branch, stop and ask for the PR number or URL.
- If Codex has already reviewed the same commit and the user did not explicitly ask to retrigger it, report that instead of posting a duplicate request.
