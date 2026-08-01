# skills

Version-controlled skills for [Claude Code](https://claude.com/claude-code) and Codex. Each lives in `<name>/SKILL.md`.

Installed skills are symlinks into this repo, so editing either path edits the same file. They follow whatever branch is checked out.

## Skills

| skill                       | purpose                                                                                    |
| --------------------------- | ------------------------------------------------------------------------------------------ |
| `babysit-pr`                | Watch a PR for bot review findings and CI, fix valid ones, repush, re-review the delta.     |
| `codex-computer-use`        | Drive a browser or desktop app through Codex, then check its screenshots yourself.          |
| `delegate`                  | Fan independent subtasks out to background subagents, keep working, shepherd them to done.  |
| `gh-rereview-latest-commit` | Ask Codex to re-review only a PR's latest commit.                                           |

## Install

```sh
ln -s "$PWD/<name>" ~/.claude/skills/<name>
ln -s "$PWD/<name>" ~/.codex/skills/<name>
```

Verify with `head -3 ~/.claude/skills/<name>/SKILL.md`. The skill shows up on the next session.
