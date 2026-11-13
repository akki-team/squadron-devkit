---
name: design
description: Directives de design de Squadron, pour l'app et pour les panneaux et réglages des extensions. À charger avant toute maquette, artifact, bloc html, page de panneau d'extension ou retouche de style dans un repo Squadron.
---

# Design Squadron

Le design a deux contrats. Repérer d'abord lequel s'applique :

- `src/app.css` et `DESIGN.md` à la racine : **l'app Squadron**.
- `extension.json` à la racine : **une extension**.

## Dans l'app

1. Lire `DESIGN.md` en entier, au moins « Règles », « Maquettes et artifacts » et la section du composant touché, puis le `:root` de `src/app.css`. `app.css` fait foi pour les valeurs, `DESIGN.md` dit à quoi elles servent.
2. Partir du CSS du composant existant le plus proche (fichier cité dans « Composants »). Réutiliser `StatusIcon`, `ModeIcon`, `Icon`, `Avatar`, `SidePanel`, `Dialog`, `Confirm`, `Select` et les classes `.btn*`, `.field`, `.option-*`, `.effect-*` sans les redéfinir.
3. Un token, une couleur en dur, une taille ou un composant visible ajouté ou changé : `DESIGN.md` est mis à jour dans le même commit. `scripts/design.test.ts` échoue sinon.

Erreurs déjà faites, à ne pas refaire :

- Rayon 8px sur une carte : c'est 6px. 8px = dialogues et cartes de conversation à barre de ton, 4px = boutons et champs.
- `--background` pris pour un fond : c'est le bleu des tâches de fond. L'encre sur cuivre est `--accent-ink`.
- Cuivre sur une action ordinaire : le cuivre signale ce qui attend l'utilisateur. Le bouton principal est clair (`.btn-primary`, `--text` sur `--ink`, 700).
- Anneau de statut redessiné à la main au lieu de `StatusIcon`.
- Contour en `border` : c'est `box-shadow: inset 0 0 0 1px …`, sauf séparateurs de zones.

## Dans une extension

- **Zones de Squadron** (items de `contributes.items`, réglages déclarés) : aucune liberté. L'extension donne un libellé `en` + `fr`, une icône du jeu de Squadron et une action ; Squadron dessine. Pas de CSS, pas de couleur.
- **Ses pages** (panneaux, page de réglages) : HTML/CSS/JS libres dans un iframe isolé, mais seules ces variables sont garanties par l'API : `--bg-main`, `--surface-1`, `--surface-2`, `--surface-3`, `--border`, `--border-focus`, `--text`, `--text-strong`, `--text-muted`, `--text-dim`, `--ink`, `--accent`, `--accent-link`, `--ok-text`, `--err-text`, `--sans`, `--mono`. Les autres tokens de l'app (`--wait-*`, `--option*`, `--surface-4`, `--menu`…) et ses classes n'y existent pas.
- Partir de `<link rel="stylesheet" href="/_squadron/kit.css" />` (fond, texte, liens, titres, `button`, `button.primary`, champs, `.muted`) et ne styler que ce qui manque, avec les variables garanties.
- Suivre quand même l'esprit des « Règles » de `DESIGN.md` de Squadron : cuivre (`--accent`) réservé à ce qui attend l'utilisateur, bouton principal clair, rayons 6/8/4px, contours en `inset box-shadow`. Lire `DESIGN.md` dans le clone local de Squadron (`../squadron/DESIGN.md`), sinon sur https://github.com/Jgrasp/squadron/blob/main/DESIGN.md.
- Pas de réseau, de `localStorage`, de popup (`alert`, `confirm`) dans la page : l'état vit dans le WASM.
- Une couleur propre à l'extension passe par `contributes.theme`, pas par du CSS en dur dans les zones de Squadron.

## Maquettes

- **Bloc ```html dans la conversation** : reçoit les tokens et les polices de l'app, pas ses classes ; pas de réseau ni de script. Recopier les règles des classes depuis `app.css`.
- **Artifact** (`$SQUADRON_ARTIFACTS`) : `<link rel="stylesheet" href="/__squadron/theme.css">` donne tout `app.css`. Ne pas recopier les tokens. Pour un panneau d'extension, n'utiliser que les variables garanties ci-dessus.
- À la taille réelle (conversation à 740px, largeurs de « Disposition » dans `DESIGN.md`).
- Plusieurs variantes côte à côte sur la même page, chacune titrée, mêmes données.
- Textes de la maquette en anglais.
- Montrer, attendre le choix, refaire. Ne rien intégrer ni committer avant un ordre explicite.
