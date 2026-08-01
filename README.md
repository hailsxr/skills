# skills

Version-controlled agent skills for [Claude Code](https://claude.com/claude-code) and Codex. Each lives in `<name>/SKILL.md`.

Install a skill by symlinking its directory into `~/.claude/skills/` or `~/.codex/skills/`. Edit either path and the version-controlled source changes with it.

## Skills

| skill                          | purpose                                                                                   |
| ------------------------------ | ----------------------------------------------------------------------------------------- |
| `delegate`                     | Fan independent subtasks out to background subagents, keep working, shepherd them to done. |
| `gh-rereview-latest-commit`    | Re-trigger GitHub Codex review for only the current PR's latest commit.                    |

## Install

```sh
ln -s "$PWD/<name>" ~/.claude/skills/<name>
ln -s "$PWD/<name>" ~/.codex/skills/<name>
```

Verify with `head -3 ~/.claude/skills/<name>/SKILL.md` or `head -3 ~/.codex/skills/<name>/SKILL.md`. The skill appears in the relevant skill list on the next session.
