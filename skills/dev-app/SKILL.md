---
name: dev-app
description: Launch Squadron's dev app (`pnpm app`) from a checkout or worktree, wait for it, hand it to Quentin or check a change yourself, in the Tauri window or in the browser bench with the mock. Load before running the dev app so Quentin can check a change, or to check it yourself.
---

# Squadron dev app

## Instances

`pnpm app` runs `scripts/app.mjs` (akki-team/squadron#46): each checkout or worktree gets its own dev app.

- Name: the worktree under `.claude/worktrees/<name>`, otherwise the checkout folder, lowercased and dashed.
- Identifier `dev.squadron.desktop.dev.<name>`: its own config, WebView data and single-instance lock. Its data lives in `~/.config/dev.squadron.desktop.dev.<name>`, seeded on first run from the installed Squadron (projects, settings, groups, layout), so a worktree's instance starts from the release's projects and settings.
- Vite: the first free port from 5173, its own cache `node_modules/.vite-<name>`.
- The name shows in the window title "Squadron dev · <name>" and in the header badge "dev · <name>". The first log line says `Squadron dev "<name>": <identifier> on http://localhost:<port>`.
- When its worktree is removed after the merge, and once the wait command's `grep` fails (no process of the instance left), delete `~/.config/dev.squadron.desktop.dev.<name>`, `~/.local/share/dev.squadron.desktop.dev.<name>` and `$XDG_RUNTIME_DIR/squadron-dev-<name>`.
- A checkout still on the old script uses plain `dev.squadron.desktop.dev` and does not collide with the new ones.

## Launch

From the checkout or worktree that holds the change, with the Bash tool's `run_in_background: true`, its output in a log file. Never `&`, `nohup` or `( … &)`: without a background task, TaskStop has nothing to stop.

```bash
timeout 3600 pnpm app > /tmp/squadron-<name>.log 2>&1
```

In a new worktree, `CARGO_TARGET_DIR=<main checkout>/src-tauri/target pnpm app` reuses the main build and saves the first full compile.

## Wait

Wait for its own process, never for a log line and never for any `target/debug/squadron`: other instances may run. Cargo starts it as the relative `target/debug/squadron`, so match its instance in its environment, not a path:

```bash
timeout 900 sh -c "until grep -qsxz 'SQUADRON_INSTANCE=<name>' \$(pgrep -f 'target/debug/[s]quadron' | sed 's|.*|/proc/&/environ|') /dev/null || grep -q ELIFECYCLE <log>; do sleep 2; done"
```

`ELIFECYCLE` in the log: the launch failed, read the log.

## Hand over

As soon as the process appears, tell Quentin it is up, with the instance name so he picks the right window: "Squadron dev · <name>".

Stop the background task with TaskStop before handing back, unless Quentin is still using it. With no task, or the process still alive, kill only this instance's:

```bash
for p in $(pgrep -f 'target/debug/[s]quadron'); do grep -qsz 'SQUADRON_INSTANCE=<name>' /proc/$p/environ && kill $p; done
```

Never a bare `pkill -f target/debug/squadron`: it kills every worktree's dev app.

## Browser bench

To check without the Tauri window: Vite without Tauri loads the mock `src/dev/mock.ts` (used when `__TAURI_INTERNALS__` is missing). Run a separate Vite with its own cache and port, in a background task:

```bash
SQUADRON_VITE_CACHE=node_modules/.vite-bench timeout 3600 pnpm exec vite --port 5181 --strictPort
```

Open `http://localhost:5181` with the chrome-devtools MCP. A temporary mock edit to reach a state is reverted before committing. Stop the task with TaskStop when done.
