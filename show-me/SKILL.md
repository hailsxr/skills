---
name: show-me
description: Bring up this worktree's dev environment, signed in, and bring the user screenshots of the change. Use when the user says "bring me a screenshot", "bring up a dev env", "show me", or "come back when it's working in the dev env". Also owns stopping a worktree's dev servers for other skills.
---

# Show me

The user runs many worktrees of the same apps at once. Everything here is scoped to **this worktree**: its ports, its processes, its screenshots.

## 1. Find what's already up

- If the user says they have it running ("I've got it up", "localhost:3000 is signed in"), use theirs and skip to step 3.
- List listeners and their working directories: `lsof -iTCP -sTCP:LISTEN -P -n`, then `lsof -a -p <pid> -d cwd -Fn` per node process. A server whose cwd is inside this worktree is yours to reuse. Anything else belongs to another worktree.

## 2. Start it

- Start the way the repo says to: its agent docs, `README`, `docs/engineering/dev-*.md`, and `package.json` scripts. Janus and Delphi run a local Convex backend plus seed data; follow their docs rather than pointing at a shared cloud deployment.
- Pick ports that are free right now for every server (Next, local Convex backend, etc.). Never take a port another worktree holds.
- Run servers with `run_in_background`, then poll until the app URL returns 200. Done when the page loads.

## 3. Sign in

Prefer the repo's dev-only login (personas). Otherwise use Google sign-in and pick the robert.hails@palyrian.com account from the chooser. Never type a password or 2FA code; stop and report if asked for one.

## 4. Capture

Use the `codex-computer-use` skill. Give Codex the URL, sign-in steps, every state to capture with a file name, and what to check in each. Save to a git-ignored folder in the worktree (`.tmp/screens/` when ignored).

- Capture in dark mode. Add light mode when the change touches colors, theming, or new surfaces.
- Cover every state the change affects: empty, filled, hover/tooltip, open menu, error, as relevant.

## 5. Look before showing

Open every screenshot yourself. Fix glitches you spot (overlaps, clipped text, wrong colors, misalignment) and recapture before presenting. Codex saying it looks right is a claim, not proof.

## 6. Present

Embed each screenshot with its absolute path (`![what it shows](/abs/path.png)`), one line of caption each. End with the URL and port, and say the server is still running.

## Stopping

When the user says "kill the dev servers" (or another skill sends you here), stop only this worktree's processes, matched by working directory:

```sh
root=$(git rev-parse --show-toplevel)
for pid in $(pgrep -f 'next|convex|vite|node'); do
  cwd=$(lsof -a -p "$pid" -d cwd -Fn 2>/dev/null | sed -n 's/^n//p')
  [[ "$cwd" == "$root"* ]] && kill "$pid"
done
```

Done when `lsof` shows no listener whose cwd is inside this worktree. Servers in other worktrees and the main checkout keep running.
