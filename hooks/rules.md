# Règles Squadron (plugin squadron-devkit)

Ce repo fait partie de Squadron (l'app ou une de ses extensions). Ces règles valent pour toute la session.

- Langues : réponses à Quentin en français. Code, identifiants, prompts envoyés à Claude, messages de commit et `CHANGELOG.md` en anglais.
- Interface traduisible : toute chaîne visible a sa version `en` et sa version `fr` (vouvoiement, « » avec U+202F à l'intérieur, U+00A0 avant `:`). Dans l'app : clés `src/lib/i18n/en` + `fr`. Dans une extension : `{ "en": …, "fr": … }` dans `extension.json`, et `CHANGELOG.fr.md` à côté de `CHANGELOG.md`.
- Pas de commentaires dans le code, sauf un piège non déductible du code, en une ligne.
- Design : avant toute maquette, artifact, bloc ```html ou retouche de style, charger le skill `squadron-devkit:design`.
- Maquettes : pendant un aller-retour de design, ne rien committer et ne rien intégrer. « On part sur B » = refaire la maquette B ; coder seulement sur ordre explicite (« intègre », « fais-le »).
- Commits : une tâche finie et vérifiée (tests, rendu) est committée sans attendre. Message d'une ligne en anglais qui décrit le résultat pour l'utilisateur. Jamais de `Co-Authored-By` ni de mention « Generated with ». Ne jamais pousser sans demande.
- Versions : une nouvelle fonctionnalité monte la mineure, des correctifs seuls montent le patch. Pour publier, charger le skill `squadron-devkit:release`.
- Extension : pour créer ou modifier une extension, charger le skill `squadron-devkit:extension`.
- Secrets : ne jamais lire ni demander un mot de passe, un jeton ou une clé. Valeurs à écrire dans un fichier : `ask_values`. Root : `run_sudo`. Passphrase SSH : `add_ssh_key`. Ne jamais créer de chemin qui rend un secret à un processus de l'agent (env, argv, stdout, fichier lisible).
- Commandes longues : les borner avec `timeout N` ; toute tâche passée en arrière-plan est arrêtée par TaskStop avant de rendre la main.
- Worktree créé à la main : `set_worktree` juste après, retiré une fois fusionné.
