# squadron-devkit

Plugin Claude Code commun aux repos de Squadron et de ses extensions.

- `hooks/rules.md` : règles injectées à chaque session (langues, i18n, commits, périmètre, versions, maquettes, secrets).
- `skills/design` : contrats de design de l'app et des pages d'extension, maquettes.
- `skills/extension` : forme d'un repo d'extension, test, publication dans le registry.
- `skills/release` : numéro de version, notes en + fr, `scripts/release.sh`.

## Installation

```bash
claude plugin marketplace add Jgrasp/claude-marketplace
claude plugin install squadron-devkit@jgrasp-marketplace
```

Puis l'activer par projet depuis Squadron.
