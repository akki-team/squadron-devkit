---
name: test-app
description: Test your own change in the Squadron app in a headless dev app you run yourself, out of Quentin's sight (`pnpm app --headless`, `scripts/drive.mjs`). Load after any UI change in the Squadron app, before telling Quentin it works.
---

# Test a Squadron change yourself

After any UI change in the Squadron app, check it here before telling Quentin it works. Ask him only for what a headless app cannot show: real mouse or keyboard feel, HiDPI, Wayland. Debug builds on Linux only (Jgrasp/squadron#49).

## Launch

From the worktree that holds the change, in a background task:

```bash
timeout 3600 pnpm app --headless > /tmp/squadron-<instance>.log 2>&1
```

`<instance>` is the worktree name. The app runs under Xvfb, with its own port, identifier `dev.squadron.desktop.dev.<instance>` and data. The first build takes a few minutes. Wait for that worktree's own process, as in the `squadron-devkit:dev-app` skill:

```bash
timeout 900 sh -c "until pgrep -f '<worktree>/src-tauri/target/debug/[s]quadron' >/dev/null || grep -q ELIFECYCLE /tmp/squadron-<instance>.log; do sleep 2; done"
```

`ELIFECYCLE`: the launch failed, read the log.

## Drive

Run `scripts/drive.mjs` from the same worktree: it finds its instance from its own checkout.

```bash
node scripts/drive.mjs run - <<'EOF'
const button = [...document.querySelectorAll('button')].find((b) => b.textContent.trim() === 'New task' || b.ariaLabel === 'New task')
button.click()
await new Promise((r) => setTimeout(r, 300))
return document.querySelector('[role=dialog]')?.textContent
EOF
```

- The script runs in the main window as the body of an async function: `return` and `await` work.
- It prints the return value as JSON, or `error <message>` and exits 1. Return text, numbers or arrays of them: a DOM node comes back as `{}`, `undefined` as `null`.
- Clicks are plain DOM calls (`button.click()`); find buttons by their text or `aria-label`.
- Synthetic pointer events (drag, resize) need `Element.prototype.setPointerCapture = () => {}` first.
- 30 s timeout by default; a third argument in ms raises it: `run script.js 60000`.

## Capture

```bash
node scripts/drive.mjs shot
```

Writes a PNG of the visible part of each open window (`main.png`, `board.png`…) in `$XDG_RUNTIME_DIR/squadron-dev-<instance>/` and prints their paths. Read them with the Read tool to see the screen. The app's output is also in `log` there.

## Stop

TaskStop the background task, then check that the same `pgrep` comes back empty: a leftover process holds the instance's lock and blocks the next launch.

## Pitfalls

- The dev app sees Quentin's real projects: never change their data during a test. No real messages to agents, no card moves on real projects.
- `xvfb-run: spawn xvfb-run ENOENT`: Xvfb is missing. Ask Quentin before installing it with `run_sudo` (`apt install xvfb`).
