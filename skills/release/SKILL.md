---
name: release
description: Publier une version de Squadron ou d'une extension Squadron — choisir le numéro, écrire les notes du changelog en anglais et en français, lancer le script de release ou pousser le tag d’une extension. À charger quand Quentin demande une release, une nouvelle version ou une mise à jour du changelog.
---

# Release Squadron

## Numéro

Lire les commits depuis le dernier tag (`git log v<dernière>..HEAD --oneline`) : une fonctionnalité → mineure suivante (0.46.2 → 0.47.0), des correctifs seuls → patch (0.46.2 → 0.46.3).

## Notes

Écrites pour l'utilisateur : ce qu'il peut faire maintenant, ou ce qui marche autrement. Pas de hash, pas de refactor, rien qu'il ne voit pas. Une ligne `- ` par changement. Version anglaise et version française avec les mêmes entrées (vouvoiement, typographie française).

## App Squadron

Les notes sont groupées par thème en `## ` (Conversation, Extensions, Updates…, comme dans `CHANGELOG.md`). Le script les descend en `### `.

```bash
scripts/release.sh <version> "<notes en>" "fr=<notes fr>"
```

Le script refuse un arbre sale, met à jour `Cargo.toml`, préfixe `CHANGELOG.md` et `CHANGELOG.fr.md`, committe `Squadron <version>`, pousse la branche `public` et le tag. Il publie : ne le lancer que sur demande explicite de Quentin.

## Extension

Voir le skill `squadron-devkit:extension`, partie « Publier ».
