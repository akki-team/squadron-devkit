---
name: extension
description: Créer, modifier, tester ou publier une extension Squadron (extension.json, logique Rust en WASM, panneaux, outils pour les agents, secrets, comptes, skills, publication dans le registry squadron-extensions). À charger dès qu'on touche un repo qui a un extension.json ou qu'on veut en créer un.
---

# Extension Squadron

Le guide fait foi : `docs/guide.md` du repo `squadron-extensions` (clone local `../squadron-extensions/docs/guide.md`, sinon https://github.com/Jgrasp/squadron-extensions/blob/main/docs/guide.md). Le lire avant d'écrire du manifeste ou du Rust : l'API évolue, ce skill ne la recopie pas.

## Forme du repo

Un repo GitHub public par extension, sur le modèle de `squadron-jira` et `squadron-google-workspace-share` :

```
extension.json        id, version, api, name { en, fr }, description { en, fr }, contributes, permissions
Cargo.toml            dépend de squadron_extension_api en git (pas en path)
Cargo.lock            committé : la CI construit en --locked
src/lib.rs
panel/                une page par panneau (index.html, .css, .js)
skills/<nom>/SKILL.md si contributes.skills
CHANGELOG.md          anglais
CHANGELOG.fr.md       mêmes sections, en français
README.md             anglais : à quoi ça sert, comment s'en servir, « What it asks Squadron for »
LICENSE
```

Nouvelle extension : copier `examples/hello` de `squadron-extensions`, remplacer la dépendance `path` par `squadron_extension_api = { git = "https://github.com/Jgrasp/squadron-extensions" }`.

## Règles

- Tout libellé visible (`name`, `description`, `label`, `title`, `hint`, `choices`) a `en` et `fr`.
- `permissions` au plus juste : chaque programme de `exec` et chaque hôte de `network` est justifié dans le README (« What it asks Squadron for »). Tout ajout redemande l'accord de l'utilisateur.
- Les secrets ne passent jamais par le WASM ni la page : `{{id}}` ou `{{account.field}}` dans un en-tête de `host::fetch`.
- La page d'un panneau suit le skill `squadron-devkit:design`, partie « Dans une extension ».
- La page parle d'abord (`squadron.send({ type: 'ready' })`), le WASM répond dans `on_event`.
- `api` du manifeste : la version d'API contre laquelle on construit. Une API plus récente que le Squadron installé ne charge pas.

## Tester

```bash
cargo build --release --target wasm32-wasip2
```

Charger le dossier dans Squadron : Settings › Extensions › Add an extension › A folder on this computer. Chaque build recharge l'extension. Une page de panneau modifiée demande de fermer et rouvrir le panneau. Pour un outil, demander à un agent d'appeler `<extension>_<outil>`. Les erreurs s'affichent sur la carte de l'extension (badge Error).

## Publier

1. Monter `version` dans `extension.json` (fonctionnalité → mineure, correctifs → patch).
2. Ajouter la section `## <version> · <YYYY-MM-DD>` en tête de `CHANGELOG.md` et de `CHANGELOG.fr.md` : une ligne `- ` par changement visible par l'utilisateur, 20 lignes et 300 caractères au plus, pas de hash ni de refactor.
3. Committer, pousser le repo de l'extension (sur demande de Quentin).
4. Dans `squadron-extensions`, une PR qui ajoute ou met à jour l'entrée de `registry.json` : `repository`, `commit` (sha complet du commit poussé), `path`, `version` identique à `extension.json` à ce commit. Une PR par extension. Ne jamais toucher `index.json` : la CI l'écrit.
