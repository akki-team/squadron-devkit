---
name: release
description: Publish a version of Squadron or of a Squadron extension — pick the number, write the changelog notes in English and French, run the release script or push an extension's tag. Load when Quentin asks for a release, a new version or a changelog update.
---

# Squadron release

## Number

Read the commits since the last version (`git fetch -q --tags origin && git log --oneline $(git describe --tags --abbrev=0 origin/main)..origin/main`). A feature → next minor (0.46.2 → 0.47.0), fixes alone → patch (0.46.2 → 0.46.3).

## Notes

Written for the user: what they can do now, or what works differently. No hash, no refactor, nothing they do not see. One `- ` line per change. English and French versions with the same entries (French uses « vous » and French typography).

## Squadron app

Notes are grouped by theme under `## ` (Conversation, Extensions, Updates…, as in `CHANGELOG.md`). The script turns them into `### `.

```bash
scripts/release.sh <version> "<en notes>" "fr=<fr notes>"
```

The script refuses a version whose `v<version>` tag already exists, and refuses if `extensions/api` differs from `../squadron-extensions/api`. It fetches `main` from GitHub into a throwaway worktree `.claude/worktrees/release-v<version>`: local work, committed or not, does not block and does not ship. There it updates `Cargo.toml`, prepends to `CHANGELOG.md` and `CHANGELOG.fr.md`, commits `Squadron <version>`, pushes it to `release/v<version>` and opens the PR "Squadron <version>" to `main`, without merging it, then deletes the worktree.

Every change reaches `main` through a PR whose required checks are `test` (build, svelte-check, vitest, `cargo fmt --check`, clippy, `cargo test`) and `secrets` (gitleaks): merge only a green PR. The only merge method is a merge commit. A push on `main` runs `test` and `build` and publishes nothing.

Quentin merges the PR. Merge it only on Quentin's explicit request. The merge starts the Tag workflow: it checks that the merged `Cargo.toml` is `<version>` and creates the `v<version>` tag on the merge commit with the `RELEASE_TAG_TOKEN` secret, and that tag starts the Release workflow (`test`, draft release, Linux, macOS and Windows builds, publishing). Only the admin can create `v*` tags. If the Release run fails, the `Squadron <version>` commit stays on `main` without a release: fix, then publish the next patch.

After the merge, wait for the Tag run, then check the tag:

```bash
gh run watch --exit-status $(gh run list --workflow Tag --branch release/v<version> --limit 1 --json databaseId --jq '.[0].databaseId')
git ls-remote --exit-code --tags origin refs/tags/v<version>
```

Only if the Tag run failed or the tag is missing, run `scripts/release.sh tag <version>`: it tags the merge commit and pushes `v<version>`.

Once the tag exists, run `scripts/release.sh watch <version>` in the background. It follows the Release run of the `v<version>` tag job by job and exits as soon as something needs Quentin: 0 when the run succeeds, 1 when a job fails or is cancelled, 2 when a job has waited more than 5 min for a runner (`QUEUE_MAX`, seconds) or run more than 15 min (`RUN_MAX`). Tell Quentin at once on 1 or 2, with the job and the run link; never wait for him to notice.

## Extension

See the `squadron-devkit:extension` skill, "Publishing" part.
