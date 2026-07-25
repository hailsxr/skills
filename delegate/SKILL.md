---
name: delegate
description: Delegate independent subtasks to background subagents and keep working while they run, monitoring for drift and intervening with corrections instead of restarts. Use when the user says "delegate", "use subagents", "parallelize", "fan out", "work in parallel", invokes /delegate, or hands over a task that splits into 2+ independent subtasks.
---

# Delegate

Split work across background subagents, keep making progress yourself, and shepherd the agents to done. Delegation is worth it only when subtasks are genuinely independent and each is big enough to outweigh a cold start — otherwise do it inline.

## 1. Split

- Carve the task into subtasks with **no shared files and no dependency on each other's results**. Anything sequential or interdependent stays on your thread.
- Reserve for yourself: the integration work, anything needing conversation context that is hard to serialize, and whatever you can finish faster than a subagent can spin up.

## 2. Pick workers

Higher = better. Cost is what the user pays; intelligence is how hard a problem the model takes unsupervised; taste covers UI/UX, code quality, API design, and copy.

| model         | cost | intelligence | taste |
| ------------- | ---- | ------------ | ----- |
| gpt-5.6-sol   | 9    | 8            | 5     |
| gpt-5.6-terra | 9    | 5            | 4     |
| sonnet-5      | 5    | 5            | 5     |
| opus-5        | 4    | 9            | 8     |

- **Bulk/mechanical** (clear-spec implementation, data analysis, migrations): gpt-5.6-sol.
- **User-facing** (UI, copy, API design): taste >= 7.
- **Reviews of plans/implementation**: opus-5, plus gpt-5.6-sol for an independent second read.
- **Never Haiku. Never fable-5** — it needs extended credits, which are off by default and stay off.
- Cost is a tie-breaker only; when axes conflict, intelligence > taste > cost. Defaults, not limits — if a cheaper model misses the bar, rerun on a smarter one without asking.

**Name every agent `<model>:<task>`**, whatever the model — `opus-5:design-settings-sheet`, `gpt-5.6-sol:review-auth`. This label (`description` on the Agent tool, `label` in workflows) is how the user tells the parallel runs apart. Use the table's names; on a codex wrapper name the real worker, not the wrapper's Claude model, since the UI already shows that. Escalating a task means relabelling it.

Reaching gpt-5.6 (Codex CLI only; the user's `~/.codex/config.toml` defaults to gpt-5.6-sol):

- `model` takes Claude models only, so wrap it: a thin `model: 'sonnet'` low-effort agent that writes a self-contained codex prompt, runs `codex exec` via Bash, and returns the report. `schema` gets structured output back.
- Codex runs can outlast Bash's 10-minute timeout — pass an explicit timeout, or background it and poll for the report file.
- Workflow budgets count Claude tokens only; codex work is invisible to `budget.spent()`.
- On "unavailable", retry after a few seconds; switch models after 3 failures.

Any two agents that might edit files concurrently get `isolation: 'worktree'` — codex implementers included, since their edits otherwise collide in the shared checkout.

## 3. Write self-contained prompts

Each subagent starts cold — missing context is the #1 failure mode. When 2+ agents need the same background, don't duplicate it into each prompt: write one **brief file** to the scratchpad (decisions made, constraints, conventions) and give every agent the path. One write, N reads, and a mid-flight correction is one file edit. Point agents at existing context (`AGENTS.md`, plan files) instead of restating it.

Every prompt then adds only what's unique to that agent:

- **Goal and definition of done** — what finished looks like, verifiable.
- **Boundaries**: files/areas it must not touch (especially its siblings' territory).
- **Return contract**: write the full report to an assigned scratchpad file; return only the file path and a ≤3-line verdict-first summary (outcome, then caveats).

## 4. Launch and keep working

- Launch all independent agents in one message, in the background (the default).
- Immediately resume your own reserved work. Never idle-wait, poll on short timers, or burn the turn narrating what the agents might be doing.

## 5. Monitor and intervene

- Check on agents at natural checkpoints between your own steps (`TaskOutput` for progress) and when completion notifications arrive.
- **Off track** (wrong scope, wrong approach, touching forbidden files): `SendMessage` a course-correction — it keeps the agent's context. Respawn only if the direction is unsalvageable; `TaskStop` first if it's actively wasting work.
- **Missing context** (agent asks, stalls, or its output shows a wrong assumption): send the missing facts the same way.
- Judge output, not price: if a cheaper model's result doesn't meet the bar, rerun with a smarter one without asking (see §2).

## 6. Integrate and report

- Triage on the verdict summaries; open an agent's full report file only when its work needs digging into. Don't pull every report into context.
- Verify each agent's work yourself before building on it — spot-check the diff, run the relevant tests. A subagent saying "done" is a claim, not evidence.
- Subagent final messages are invisible to the user: relay outcomes in your summary — what each agent did, what you verified, what you corrected or redid. Name agents by their labels so the summary matches what the user watched run.
