# PR-Agent — notes & doc interne

Ce dossier rassemble tout ce qu'on a appris et fait pendant le POC PR-Agent du **27/04/2026**, pour LogiSpear.

## TL;DR

- **PR-Agent** est un bot open source (AGPL-3.0) qui review automatiquement les Merge Requests sur GitLab, GitHub et Bitbucket.
- On l'a déployé sur **notre Coolify**, à côté de **GitLab CE** auto-hébergé.
- Il appelle un LLM (Claude Haiku via OpenRouter dans notre cas) pour faire la review.
- Coût constaté : **0,06€ pour 7 requêtes** (≈ 1 centime / review).
- Déclenché automatiquement par webhook à chaque ouverture de MR.
- Résultat du POC : il a chopé les 2 vrais bugs qu'on avait glissés volontairement dans le code de test.

## Fichiers

- [architecture.md](./architecture.md) — comment ça marche (acteurs, flow, schéma)
- [deploiement.md](./deploiement.md) — étapes du POC, reproductible
- [alternatives.md](./alternatives.md) — comparaison avec CodeRabbit et GitLab Duo
- [commandes.md](./commandes.md) — commandes manuelles dispos (`/review`, `/improve`, `/ask`, etc.)

## Ce qu'on a montré pendant le POC

- Review automatique sans intervention humaine (30 sec après ouverture de la MR)
- Description de MR auto-générée
- Label `Review effort N/5` auto-posé
- Commandes manuelles utilisables en commentaire
- Tout sur notre infra, aucune donnée ne sort sauf l'appel LLM vers OpenRouter

## Liens utiles

- Repo GitHub : https://github.com/qodo-ai/pr-agent
- Doc officielle : https://qodo-merge-docs.qodo.ai/
- Image Docker utilisée : `codiumai/pr-agent:gitlab_webhook`
