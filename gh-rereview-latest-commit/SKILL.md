---
name: gh-rereview-latest-commit
description: Ask Codex on GitHub to re-review only a PR's latest commit. Use when the user asks to retrigger a Codex review or re-review recent fixes.
---

# Re-review Latest PR Commit

Post one `@codex review` comment scoped to the PR's latest commit. Don't change code, push, resolve threads, or wait for the review unless asked.

1. Find the PR with `gh pr view`. If the branch has none, ask for the number.
2. Take the full remote `headRefOid` as the target. If it differs from local `HEAD`, mention it, but still target the remote head unless the user meant an unpushed commit.
3. If Codex already reviewed that commit and the user didn't ask to retrigger, say so instead of posting a duplicate.
4. Pick a focus from the request (e.g. "the recent review fixes"), defaulting to "the changes introduced by the latest commit". Post it with `gh pr comment --body-file` so the text is never run as shell:

   ```text
   @codex review

   Review only the latest commit, `<full-head-sha>`, relative to its first parent. Focus specifically on <focus>. Do not re-review earlier commits or the full pull request diff.
   ```

   Always use the full SHA so the scope holds if more commits land.

5. Return the comment URL and short SHA. Say the review was triggered, not that it finished.
