---
name: extension
description: Create, change, test or publish a Squadron extension (extension.json, Rust logic in WASM, panels, tools for agents, secrets, accounts, skills, publishing to the squadron-extensions registry). Load as soon as you touch a repo with an extension.json or want to create one.
---

# Squadron extension

The guide is authoritative: `docs/guide.md` in the `squadron-extensions` repo (local clone `../squadron-extensions/docs/guide.md`, otherwise https://github.com/Jgrasp/squadron-extensions/blob/main/docs/guide.md). Read it before writing manifest or Rust: the API evolves, this skill does not copy it.

## Repo shape

One public GitHub repo per extension, modeled on `squadron-jira` and `squadron-google-workspace-share`:

```
extension.json        id, version, api, name { en, fr }, description { en, fr }, contributes, permissions
Cargo.toml            depends on squadron_extension_api via git (not path)
Cargo.lock            committed: CI builds with --locked
src/lib.rs
panel/                one page per panel (index.html, .css, .js)
skills/<name>/SKILL.md if contributes.skills
CHANGELOG.md          English
CHANGELOG.fr.md       same sections, in French
README.md             English: what it is for, how to use it, "What it asks Squadron for"
LICENSE
```

New extension: copy `examples/hello` from `squadron-extensions`, replace the `path` dependency with `squadron_extension_api = { git = "https://github.com/Jgrasp/squadron-extensions" }`.

## Rules

- Every visible label (`name`, `description`, `label`, `title`, `hint`, `choices`) has `en` and `fr`.
- `permissions` kept to the minimum: each `exec` program and each `network` host is justified in the README ("What it asks Squadron for"). Any addition asks the user's approval again.
- Secrets never go through the WASM or the page: `{{id}}` or `{{account.field}}` in a `host::fetch` header.
- A panel page follows the `squadron-devkit:design` skill, "In an extension" part.
- The page speaks first (`squadron.send({ type: 'ready' })`), the WASM answers in `on_event`.
- `api` in the manifest: the API version you build against. An API newer than the installed Squadron does not load.

## Testing

```bash
cargo build --release --target wasm32-wasip2
```

Load the folder in Squadron: Settings › Extensions › Add an extension › A folder on this computer. Each build reloads the extension. A changed panel page needs the panel closed and reopened. For a tool, ask an agent to call `<extension>_<tool>`. Errors show on the extension's card (Error badge).

## Publishing

1. Bump `version` in `extension.json` (feature → minor, fixes → patch).
2. Add the `## <version> · <YYYY-MM-DD>` section at the top of `CHANGELOG.md` and `CHANGELOG.fr.md`: one `- ` line per user-visible change, at most 20 lines and 300 characters, no hash or refactor.
3. Deliver it through a PR. Once it is merged, on Quentin's request, push the tag `v<version>` (`<id>/v<version>` if the extension is in a subfolder) on the merged commit. An extension using a newer API declares `"squadron": ">=<Squadron version>"`.
4. The `squadron-extensions` CI (Registry workflow, hourly, or right away with `gh workflow run Registry --repo Jgrasp/squadron-extensions`) builds the newest tag, checks the id, version and changelog section, publishes the archive and adds it to `index.json`. Then check the version in `index.json`; a rejected tag is in `rejected.json` with its reason.
5. New extension only: a PR in `squadron-extensions` that adds `"<id>": { "repository": "https://github.com/…" }` to `registry.json`, with `"path"` if `extension.json` is not at the root. No PR per version. Never touch `index.json`: CI writes it.
