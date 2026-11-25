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
- `permissions` kept to the minimum: each `exec` program (and whether it has `"secrets": true`) and each `network` host is justified in the README ("What it asks Squadron for"). Any addition asks the user's approval again.
- Values the user must copy, such as the scopes of a token, go in `fields[].copy` of `contributes.accounts` (shown as copy buttons under the hint), not in `hint`; they are plain strings, exempt from the en/fr rule.
- A panel page follows the `squadron-devkit:design` skill, "In an extension" part.
- The page speaks first (`squadron.send({ type: 'ready' })`), the WASM answers in `on_event`.
- `api` in the manifest is `1`. The API version is the `squadron_extension_api` crate's: built against an API newer than the installed Squadron, the extension does not load, so declare the oldest Squadron that has it in `squadron` (table at the end of the guide's "Limits"; API 1.16.0 → `">=0.64.0"`).

## Secrets

The guide's "Secrets and network", "Secrets for a program", "Tokens from a service" and "Opening the account form" are authoritative.

- A secret (password, API token, OAuth token, client secret) never goes through the WASM, a panel, extension state, a file, a program's `args` or `env`. It lives in the keyring as a `contributes.secrets` entry or a `secret` / `captured` account field. The extension writes `{{secret-id}}` or `{{<account>.<field>}}` (`{{base64:…}}` for one or two parts) and Squadron resolves it only in a `host::fetch*` header, a form-urlencoded body, and the stdin of `host::exec_input` for a program declared `{ "program": "…", "secrets": true }` in `permissions.exec`. Never in a URL, a JSON body, `args` or `env`.
- Password to a program (database client): one account per connection, the password a `secret` field, the server a `text` field (fields of an account with a `host` field never reach a program). `"permissions": { "exec": [{ "program": "java", "secrets": true }] }`, then `host::exec_input` with `{{base64:<account>.password}}` in the JSON input, decoded by the program: values are written without JSON escaping. Never a pgpass, `.netrc`, `PGPASSWORD`, nor a password in args, env or state; a client that only reads it that way cannot be used, run one that reads stdin.
- OAuth tokens: `refresh-token` and `access-token` are `captured` account fields, filled by `host::fetch_capture` (the WASM only sees `[secret]`); the client secret is in `contributes.secrets`. The refresh is a form body with `{{client-secret}}` and `{{<account>.refresh-token}}`, and captures `access_token` again (a plain fetch would return it in clear). Calls send `Bearer {{<account>.access-token}}`; on a 401, refresh and resend. Never a token in state, a file or a panel; only the single-use authorization code goes from a panel to the WASM. With `"check": true`, `check_account` answers `Ok` on the "not set" error of a captured field, or the form never saves.
- `fetch_capture` needs a saved account: `host::open_account(None | Some(id), prefill)` opens the account form in Settings and returns a form id at once; the `account.form` event then brings `{ form, account }` or `{ form, error }`. Never wait for it in a loop.
- A tool `command` still gets `contributes.secrets` in its env: new code does not use it.
- These features are API 1.16.0: `"squadron": ">=0.64.0"`.

## Testing

```bash
cargo build --release --target wasm32-wasip2
```

Load the folder in Squadron: Settings › Extensions › Add an extension › A folder on this computer. Each build reloads the extension. A changed panel page needs the panel closed and reopened. For a tool, ask an agent to call `<extension>_<tool>`. Errors show on the extension's card (Error badge).

## Publishing

1. Bump `version` in `extension.json` (feature → minor, fixes → patch).
2. Add the `## <version> · <YYYY-MM-DD>` section at the top of `CHANGELOG.md` and `CHANGELOG.fr.md`: one `- ` line per user-visible change, at most 20 lines and 300 characters, no hash or refactor.
3. Deliver it through a PR. Once it is merged, on Quentin's request, push the tag `v<version>` (`<id>/v<version>` if the extension is in a subfolder) on the merged commit.
4. The `squadron-extensions` CI (Registry workflow, hourly) builds the newest tag, checks the id, version and changelog section, publishes the archive and adds it to `index.json`. Do not run the workflow yourself: wait for it, or if Quentin wants it now, propose an agent on `squadron-extensions` with `start_agent`. Then check the version in `index.json`; a rejected tag is in `rejected.json` with its reason, to hand to that same kind of agent if the cause is on the registry side.
5. New extension only: `registry.json` in `squadron-extensions` needs `"<id>": { "repository": "https://github.com/…" }`, with `"path"` if `extension.json` is not at the root. Propose an agent on `squadron-extensions` with `start_agent` to add it through its PR; never edit that repo yourself. No entry per version. `index.json` is written by CI only.
