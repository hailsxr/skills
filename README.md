# skills

Version-controlled [Claude Code](https://claude.com/claude-code) skills. Each lives in `<name>/SKILL.md`, mirroring the layout of `~/.claude/skills/`.

Claude Code discovers skills in `~/.claude/skills/`, so the installed skill is a symlink to this repo — edit either path, it's the same file.

## Skills

| skill      | purpose                                                                                          |
| ---------- | ------------------------------------------------------------------------------------------------ |
| `delegate` | Fan independent subtasks out to background subagents, keep working, shepherd them to done.        |

## Install

```sh
ln -s "$PWD/<name>" ~/.claude/skills/<name>
```

Verify with `head -3 ~/.claude/skills/<name>/SKILL.md`; the skill shows up in Claude Code's skill list on the next session.
