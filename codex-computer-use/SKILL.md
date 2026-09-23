---
name: codex-computer-use
description: Drive a browser or desktop app through Codex (`codex exec`) and check its screenshots yourself. Use when you need to see or click through a real UI. Prefer it over Claude's own browser tools.
---

# Codex Computer Use

Codex is better at computer use than Claude. You plan the check and judge the result; Codex does the clicking. Run it straight from Bash, no subagent.

Skip it if tests, `curl`, or reading the code can answer the question.

## Prompt

Codex starts cold, so the prompt has to stand alone:

- What to open (URL or app), the steps, and exactly what to check. The dev server is probably already running — find its URL first.
- Save a screenshot of every checked state to `<outdir>` (a folder in your scratchpad).
- Never enter real credentials, pay, send messages, or delete real data. Stop and report if it hits a login or a destructive step.
- Report: verdict first (pass / fail / blocked), then one line per finding with its screenshot path.

## Run

```sh
codex exec -s workspace-write --skip-git-repo-check \
  -C <project> --add-dir <outdir> \
  -o <outdir>/report.md "<prompt>" < /dev/null
```

- `< /dev/null` is required or it hangs waiting on stdin.
- Use `run_in_background` and keep working. You'll be notified when it's done.
- Defaults to `gpt-6-sol`. Add `-m gpt-6-astra` only if Sol fails a tricky flow.

## Verify

Read the report, then open the key screenshots and look yourself. Codex saying it looks right is a claim, not proof. Design calls are yours: GPT has weak taste.

After a fix, re-check in the same session so Codex keeps its context. The session id is in the run's header:

```sh
codex exec resume <session-id> -o <outdir>/report-2.md "<what changed; re-check X>" < /dev/null
```
