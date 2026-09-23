---
name: codex-computer-use
description: Hand browser and desktop computer use to Codex (GPT-6) via `codex exec`, then verify its screenshots yourself. Use whenever a task needs a real UI seen or driven — checking a change in the browser, clicking through a flow, reproducing a visual bug, screenshotting an app, or operating a desktop app — or the user says "computer use", "check it in the browser", or "click through it". Prefer this over Claude's own browser tools.
---

# Codex Computer Use

Codex is much better at computer use than Claude, so Claude plans and judges while Codex drives. Codex in `codex exec` mode has Unified Computer Use (browser tabs and desktop apps) plus agent-browser tools. Run it straight from Bash; no subagent needed.

Skip it when a cheaper check answers the question: tests, `curl`, a type check, or reading the code.

## 1. Prep

- Assume the dev server is already running; find its URL (check listening ports or the project's dev script) before starting one.
- Make an output dir in the scratchpad, e.g. `<scratchpad>/cu-<task>/`, for screenshots and the report.

## 2. Write the prompt

Codex starts cold. The prompt must stand alone:

- **Goal and done**: what to open (URL or app), the steps to take, and exactly what to check.
- **Evidence**: save a screenshot of every checked state to the output dir, with descriptive filenames.
- **Guardrails**: never enter real credentials, pay, send messages, or delete data outside local dev. If blocked by a login or a destructive step, stop and report instead of working around it.
- **Report**: verdict first (pass / fail / blocked), then one line per finding with its screenshot path.

## 3. Run

```sh
codex exec -s workspace-write --skip-git-repo-check \
  -C <project> --add-dir <outdir> \
  -o <outdir>/report.md "<prompt>" < /dev/null
```

- `< /dev/null` is required, or exec waits on stdin.
- Runs often take minutes: use Bash with `run_in_background` and keep working. The harness tells you when it's done, so don't poll.
- The config default is `gpt-6-sol`. Add `-m gpt-6-astra` only for long, tricky flows Sol failed.
- Label the run `gpt-6-sol:<task>` in your updates so it matches the delegate naming.

## 4. Verify

- Read `report.md`, then open the key screenshots with Read and look yourself. Codex saying "looks right" is a claim, not evidence.
- Judge design yourself: GPT models are weak on taste, so spacing, alignment, theming, and polish issues are yours to catch.

## 5. Iterate

After fixing something, re-check in the same session so Codex keeps its context (open tabs, the path through the app). The session id is printed in the run's header:

```sh
codex exec resume <session-id> -o <outdir>/report-2.md "<what changed; re-check X>" < /dev/null
```

Tell the user what was checked, the verdict, and what you saw in the screenshots, including anything you disagreed with Codex on.
