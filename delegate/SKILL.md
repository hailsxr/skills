---
name: delegate
description: Hand independent subtasks to background subagents, keep working, and steer them to done. Use when the user says "delegate", "use subagents", "parallelize", "fan out", or "work in parallel".
---

# Delegate

Split work across background subagents while you keep working. Only worth it when subtasks are truly independent and each is big enough to beat a cold start. Otherwise do it inline.

## 1. Split

- Subtasks must share no files and not depend on each other's results. Anything sequential stays with you.
- Keep for yourself: integration, anything that needs this conversation's context, and anything you can finish faster than an agent can spin up.

## 2. Pick workers

Higher = better. Cost is what the user pays, intelligence is how hard a problem it handles alone, taste is UI/UX, code quality, API design, and copy.

| model       | cost | intelligence | taste |
| ----------- | ---- | ------------ | ----- |
| gpt-6-sol   | 8    | 8            | 5     |
| gpt-6-astra | 5    | 9            | 6     |
| opus-5.5    | 4    | 7            | 8     |
| sonnet-5    | 5    | 3            | 5     |

- **Default**: gpt-6-sol. Implementation, migrations, data work, backend.
- **Hard or high-stakes non-UI work**, or Sol missed: gpt-6-astra.
- **Mainly UI/UX or user-facing copy**: opus-5.5. Small UI bits inside a GPT task stay with that agent. You catch design issues when integrating.
- **Reviews**: gpt-6-sol first (astra if high-stakes), then opus-5.5 for quality and taste.
- **sonnet-5** is only the codex wrapper. Never give it real work.
- **Never Haiku. Never Fable** (5 or 5.1): it only gets 50% of the weekly limit.
- When scores conflict: intelligence > taste > cost. If a cheaper model's work isn't good enough, rerun on a smarter one without asking.

**Label every agent `<model>:<task>`**, e.g. `opus-5.5:design-settings-sheet`, `gpt-6-sol:review-auth`. It's the `description` on the Agent tool or `label` in workflows, and it's how the user tells runs apart. For codex wrappers, name the GPT model, not the wrapper. Relabel when you escalate.

**Reaching GPT** (Codex CLI only; defaults to gpt-6-sol, use `-m gpt-6-astra` for Astra):

- `model` only takes Claude models, so wrap it: a low-effort `model: 'sonnet'` agent writes a self-contained prompt, runs `codex exec` via Bash, and returns the report. Use `schema` for structured output.
- Run with `< /dev/null` (or it hangs on stdin) and `-o <file>` to capture the final message.
- Runs can outlast Bash's 10-minute timeout. Set a timeout, or background it and wait for the report file.
- Workflow budgets only count Claude tokens. Codex work doesn't show in `budget.spent()`.
- On "unavailable", retry after a few seconds. Switch models after 3 failures.

Agents that might edit files at the same time get `isolation: 'worktree'`, codex ones included.

## 3. Write prompts

Agents start cold, and missing context is the #1 failure. If 2+ agents need the same background, write it once to a brief file in the scratchpad and pass the path. A mid-run correction is then one edit. Point to existing docs (`AGENTS.md`, plan files) instead of restating them.

Each prompt then adds:

- **Goal and done**: what finished looks like, checkably.
- **Boundaries**: files it must not touch, especially other agents' areas.
- **Return**: full report to an assigned scratchpad file; reply with just the path and a ≤3-line summary, verdict first.

## 4. Launch and monitor

- Launch all agents in one message, in the background. Go straight back to your own work. No idle waiting, short polling, or narrating.
- Check in between your own steps (`TaskOutput`) and when agents finish.
- **Off track or missing context**: `SendMessage` a correction or the missing facts. It keeps the agent's context. Respawn only if it's unsalvageable, and `TaskStop` it first if it's wasting work.

## 5. Integrate and report

- Triage on the summaries. Only open full reports that need digging into.
- Verify before building on anything: check the diff, run the tests. "Done" is a claim, not proof.
- The user can't see agent replies. Summarize what each did, what you checked, and what you fixed, using their labels.
