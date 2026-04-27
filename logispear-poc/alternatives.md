# Alternatives à PR-Agent

Récap des 3 options évaluées pour la review automatique de MR sur GitLab self-managed.

## 1. PR-Agent (Qodo) — celui qu'on a choisi

- **License** : AGPL-3.0 (open source, vrai)
- **Hébergement** : self-hosted (Docker)
- **Coût** : juste les tokens LLM (~1 centime/review avec Claude Haiku)
- **Repo** : https://github.com/qodo-ai/pr-agent (~11k stars, repo actif)

Pour :
- Pas cher du tout
- Contrôle total des données
- Pas de SaaS externe (sauf l'appel LLM, qu'on peut router où on veut)
- Multi-LLM (on peut switcher facilement de modèle)
- Multi-provider Git (GitLab, GitHub, Bitbucket)

Contre :
- Faut maintenir l'instance (mais c'est juste un conteneur Docker)
- Pas de support pro (mais la doc est bonne et la communauté active)

## 2. CodeRabbit

- **License** : propriétaire
- **Hébergement** :
  - Cloud : SaaS uniquement
  - Self-hosted : possible mais réservé à l'offre Enterprise
- **Coût** :
  - Cloud Pro : $24/user/mois (supporte GitLab self-managed via OAuth)
  - Self-Hosted Enterprise : ~$15 000/mois minimum (à partir de 500 users)

Pour :
- Produit poli, UI soignée
- Support pro

Contre :
- Pas open source
- Cher dès qu'on veut self-héberger
- En mode cloud, le code passe par leur infra (problématique pour nos contraintes data)

## 3. GitLab Duo Code Review

- **License** : propriétaire (intégré à GitLab)
- **Hébergement** : natif dans GitLab
- **Coût** :
  - Forfait : Ultimate $99/user/mois + Duo Enterprise $39/user/mois = **$138/user/mois**
  - Ou pay-per-review : $0,25/review via GitLab Credits

Pour :
- Zéro setup, intégré nativement
- Stack 100% GitLab

Contre :
- Très cher en forfait (multiplier par le nombre de devs)
- Le pay-per-review devient vite plus cher que PR-Agent (0,25$ vs ~0,01€ par review)
- Dépend de leur LLM, pas de choix du modèle

## Pourquoi PR-Agent l'emporte pour LogiSpear

| Critère | PR-Agent | CodeRabbit | GitLab Duo |
|---|---|---|---|
| Open source vrai | ✅ | ❌ | ❌ |
| Self-hosted gratuit | ✅ | ❌ ($15k/mois) | ❌ |
| Données qui restent chez nous | ✅ (sauf LLM) | ❌ | ❌ |
| Coût raisonnable | ✅ (~1ct/MR) | 💸 $24/user/mois | 💸💸 $138/user/mois |
| Choix du LLM | ✅ | ❌ | ❌ |
| Qualité de la review | ✅ (validée au POC) | ✅ | ✅ |

Verdict : **PR-Agent** coche toutes les cases qui comptent pour nous (open source, self-hosted, contrôle data, prix), avec une qualité de review confirmée au POC.
