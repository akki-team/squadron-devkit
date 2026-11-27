# squadron-devkit

Claude Code plugin shared by the Squadron repos and its extensions.

- `hooks/rules.md`: rules injected at every session (languages, i18n, branches, commits, scope, versions, mockups, secrets, PRs, dev app).
- `skills/dev-app`: launching and checking Squadron's dev app, one instance per checkout or worktree, browser bench with the mock.
- `skills/test-app`: testing a change yourself in a headless dev app, driven and captured with `scripts/drive.mjs`.
- `skills/design`: design contracts for the app and extension pages, mockups.
- `skills/extension`: shape of an extension repo, secrets and accounts, testing, publishing to the registry.
- `skills/release`: version number, en + fr notes, `scripts/release.sh`.

## Installation

Enable it per project from Squadron.
