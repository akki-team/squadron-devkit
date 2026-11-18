---
name: release
description: Publish a version of Squadron or of a Squadron extension — pick the number, write the changelog notes in English and French, run the release script or push an extension's tag. Load when Quentin asks for a release, a new version or a changelog update.
---

# Squadron release

## Number

Read the commits since the last local version (`git log --oneline $(git log -1 --format=%h --grep='^Squadron [0-9]')..HEAD`): the `v<version>` tags are on the public history, not on local `main`. A feature → next minor (0.46.2 → 0.47.0), fixes alone → patch (0.46.2 → 0.46.3).

## Notes

Written for the user: what they can do now, or what works differently. No hash, no refactor, nothing they do not see. One `- ` line per change. English and French versions with the same entries (French uses « vous » and French typography).

## Squadron app

Notes are grouped by theme under `## ` (Conversation, Extensions, Updates…, as in `CHANGELOG.md`). The script turns them into `### `.

```bash
scripts/release.sh <version> "<en notes>" "fr=<fr notes>"
```

The script works in a throwaway worktree `.claude/worktrees/release-v<version>` of the project, from the latest `main` commit: other agents' uncommitted work does not block and does not ship. There it updates `Cargo.toml`, prepends to `CHANGELOG.md` and `CHANGELOG.fr.md`, commits `Squadron <version>`, pushes the snapshot to `release/v<version>` and opens the PR "Squadron <version>" to `main`, without merging it. It brings the commit back to local `main` and deletes the worktree. It refuses if `extensions/api` differs from `../squadron-extensions/api`.

GitHub only receives versions: never `git push` local `main`, whose history differs. `main` on GitHub refuses force pushes and deletion, and only accepts a PR merged by rebase (a squash would add "(#N)" to the message and break the release).

Quentin merges the PR: that is what starts the release. Merge it only on Quentin's explicit request. Then `scripts/release.sh tag` sets the `v<version>` tag locally on the merged commit and updates the `public` branch; the next run of the script also does it for the previous version.

On GitHub, the `test` job of the Release workflow (build, svelte-check, vitest, `cargo fmt --check`, clippy, `cargo test`) runs on the PR: merge only a green PR. The merge runs `test` again, then the draft release, the Linux, macOS and Windows builds, and publishing. If it fails, the `Squadron <version>` commit stays on `main` without a release: fix, then publish the next patch.

Right after the PR opens, and again right after the merge, run `scripts/release.sh watch <version>` in the background. It follows the Release run of that version job by job and exits as soon as something needs Quentin: 0 when the run succeeds, 1 when a job fails or is cancelled, 2 when a job has waited more than 5 min for a runner (`QUEUE_MAX`, seconds) or run more than 15 min (`RUN_MAX`). Tell Quentin at once on 1 or 2, with the job and the run link; never wait for him to notice.

## Extension

See the `squadron-devkit:extension` skill, "Publishing" part.
