---
name: release
description: Publier une version de Squadron ou d'une extension Squadron — choisir le numéro, écrire les notes du changelog en anglais et en français, lancer le script de release ou pousser le tag d’une extension. À charger quand Quentin demande une release, une nouvelle version ou une mise à jour du changelog.
---

# Release Squadron

## Numéro

Lire les commits depuis la dernière version locale (`git log --oneline $(git log -1 --format=%h --grep='^Squadron [0-9]')..HEAD`) : les tags `v<version>` sont sur l'historique public, pas sur `main` local. Une fonctionnalité → mineure suivante (0.46.2 → 0.47.0), des correctifs seuls → patch (0.46.2 → 0.46.3).

## Notes

Écrites pour l'utilisateur : ce qu'il peut faire maintenant, ou ce qui marche autrement. Pas de hash, pas de refactor, rien qu'il ne voit pas. Une ligne `- ` par changement. Version anglaise et version française avec les mêmes entrées (vouvoiement, typographie française).

## App Squadron

Les notes sont groupées par thème en `## ` (Conversation, Extensions, Updates…, comme dans `CHANGELOG.md`). Le script les descend en `### `.

```bash
scripts/release.sh <version> "<notes en>" "fr=<notes fr>"
```

Le script travaille dans un worktree jetable `.worktrees/release-v<version>` du projet, à partir du dernier commit de `main` : le travail non commité d'autres agents ne bloque pas et ne part pas. Il y met à jour `Cargo.toml`, préfixe `CHANGELOG.md` et `CHANGELOG.fr.md`, committe `Squadron <version>`, pousse l'instantané sur `release/v<version>` et ouvre la PR « Squadron <version> » vers `main`, sans la fusionner. Il ramène le commit sur `main` local et supprime le worktree. Il refuse si `extensions/api` diffère de `../squadron-extensions/api`.

GitHub ne reçoit que les versions : jamais de `git push` de `main` local, dont l'historique diffère. `main` sur GitHub refuse le push forcé et la suppression, et n'accepte qu'une PR fusionnée en rebase (un squash ajouterait « (#N) » au message et casserait la release).

Quentin fusionne la PR : c'est ce qui lance la release. Ne la fusionner que sur sa demande explicite. Ensuite, `scripts/release.sh tag` pose le tag `v<version>` en local sur le commit fusionné et met à jour la branche `public` ; le lancement suivant du script le fait aussi pour la version précédente.

Sur GitHub, le workflow Release se lance à chaque push sur `main` dont le message commence par « Squadron » : le job `test` (build, svelte-check, vitest, `cargo test`) passe d'abord, puis la release brouillon, les builds Linux, macOS et Windows, et la publication. S'il échoue, le commit `Squadron <version>` reste sur `main` sans release : corriger, puis publier le patch suivant.

## Extension

Voir le skill `squadron-devkit:extension`, partie « Publier ».
