# Commandes manuelles PR-Agent

Une fois PR-Agent branché sur la MR, on peut lui parler en commentaire. Une commande par commentaire, attendre la réponse avant la suivante.

## Les commandes principales

### `/review`

Relance une review complète de la MR (PR Reviewer Guide).
Utile quand on a poussé un nouveau commit et qu'on veut une nouvelle analyse.

### `/improve`

Génère des suggestions d'amélioration de code, classées par catégorie (Possible issue, General) et impact (Low/Medium/High).
C'est la commande la plus actionable pour le dev.

### `/ask <question>`

Pose une question libre en langage naturel sur la MR. PR-Agent analyse le code et répond.

Exemples :
- `/ask Pourquoi multiply ne gère pas les nombres négatifs ?`
- `/ask Cette modification casse-t-elle la rétrocompat ?`
- `/ask Y a-t-il des tests à ajouter pour ce changement ?`

### `/describe`

Regénère la description de la MR (PR Type, bullet points résumant les changements).
Utile si on a foiré la description initiale ou si on a changé de scope.

### `/help`

Liste toutes les commandes dispos avec leur syntaxe.

## Commandes plus avancées (optionnelles)

### `/update_changelog`

Met à jour le CHANGELOG.md à partir des changements de la MR.

### `/add_docs`

Suggère des docstrings et commentaires à ajouter au code modifié.

### `/test`

Suggère des tests unitaires pour les changements de la MR.

### `/similar_issue`

Cherche des MR ou issues passées qui ressemblent à celle en cours.

## Bon à savoir

- Toutes les commandes sont déclenchées via le webhook (event `Comments`).
- Les commandes ne consomment des tokens LLM que quand on les invoque (pas en permanence).
- On peut configurer la review automatique pour qu'elle se déclenche **uniquement** sur les MR taggées avec un label spécifique, si on veut limiter le coût ou la verbosité.
