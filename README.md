# squadron-devkit

Claude Code plugin shared by the Squadron repos and its extensions.

- `hooks/rules.md`: rules injected at every session (languages, i18n, branches, commits, scope, versions, mockups, secrets, PRs, dev app).
- `skills/dev-app`: launching and checking Squadron's dev app, one instance per checkout or worktree, browser bench with the mock.
- `skills/design`: design contracts for the app and extension pages, mockups.
- `skills/extension`: shape of an extension repo, testing, publishing to the registry.
- `skills/release`: version number, en + fr notes, `scripts/release.sh`.

## Installation

```bash
claude plugin marketplace add Jgrasp/claude-marketplace
claude plugin install squadron-devkit@jgrasp-marketplace
```

Then enable it per project from Squadron.
