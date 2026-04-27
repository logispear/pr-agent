# Comment ça marche

## Les 4 acteurs en jeu

| Acteur | Rôle | Où ça tourne |
|---|---|---|
| **GitLab CE** | Héberge le code et les MR | Conteneur Docker sur notre Coolify |
| **PR-Agent** | Le bot qui review et commente | Conteneur Docker sur notre Coolify |
| **OpenRouter** | Routeur LLM (accès à Claude, GPT, etc.) | SaaS externe |
| **Coolify** | Orchestre les conteneurs Docker | Notre serveur (185.158.132.58) |

## Le flow quand on ouvre une MR

1. Dev ouvre une MR sur GitLab.
2. GitLab envoie un webhook (HTTP POST automatique) à l'URL de PR-Agent.
3. PR-Agent vérifie le `Secret Token` partagé pour confirmer que c'est bien GitLab qui appelle.
4. PR-Agent récupère le diff et le code via l'API GitLab (avec son `Personal Access Token`).
5. Il construit un prompt structuré et l'envoie à OpenRouter.
6. OpenRouter route vers Claude Haiku, qui renvoie une analyse.
7. PR-Agent reposte la review en commentaire sur la MR via l'API GitLab.

Durée totale : environ 30 secondes.

## Schéma simplifié

```
[Dev] → ouvre MR → [GitLab CE]
                       │
                       │ webhook (POST /webhook)
                       ▼
                  [PR-Agent] ─── API GitLab ──→ récupère diff + contexte
                       │
                       │ HTTPS
                       ▼
                  [OpenRouter] ──→ [Claude Haiku]
                       │
                       │ analyse
                       ▼
                  [PR-Agent] ─── API GitLab ──→ poste commentaires
```

Tout vit chez nous sauf la dernière étape (l'appel LLM sort vers OpenRouter, qui parle ensuite à Anthropic).

## Mental model en 1 phrase

> GitLab héberge le code et envoie un signal à chaque MR. PR-Agent reçoit ce signal, relit le code, demande à Claude (via OpenRouter) ce qu'il en pense, et reposte la réponse en commentaire.

## Les 3 clés et à quoi elles servent

### GitLab Personal Access Token (`glpat-...`)

La carte d'identité de PR-Agent côté GitLab. Sans ça, il peut ni lire le code ni poster de commentaires.

- Scope nécessaire : `api`
- Généré dans : User Settings (avatar) → Access Tokens
- Stocké dans la variable d'env : `GITLAB__PERSONAL_ACCESS_TOKEN`

### Webhook Secret Token

Un mot de passe partagé, identique des deux côtés (PR-Agent + config webhook GitLab). Empêche n'importe qui sur le net de balancer des fausses requêtes à PR-Agent.

- Choisi par nous (n'importe quelle chaîne aléatoire)
- Stocké dans la variable d'env : `GITLAB__SHARED_SECRET`
- Renseigné aussi dans GitLab → Settings → Webhooks → champ "Secret token"

### OpenRouter API Key (`sk-or-v1-...`)

C'est elle qui se fait débiter à chaque review. Sur Claude Haiku, ~1 centime par review.

- Récupérée sur openrouter.ai → Keys
- Crédit minimum à acheter : $5
- Stockée dans la variable d'env : `OPENROUTER__KEY`

Les valeurs réelles vivent uniquement dans les variables d'environnement Coolify du service `pr-agent`.

## Pourquoi OpenRouter et pas direct Anthropic

OpenRouter donne accès à plein de modèles (Claude, GPT, Llama, etc.) avec une seule clé. On peut switcher de modèle juste en changeant `CONFIG__MODEL`, sans recréer de compte. PR-Agent supporte aussi direct Anthropic, OpenAI, Azure, etc. — on a juste choisi OpenRouter parce qu'on avait déjà la clé.

## Webhook : c'est quoi exactement

Un webhook c'est juste **un HTTP POST automatique**. GitLab dit : "à chaque fois qu'il se passe X événement (ouverture de MR, nouveau commentaire, etc.), j'envoie un POST à cette URL avec les détails de l'événement". PR-Agent expose une URL `/webhook`, GitLab tape dessus, PR-Agent fait son taf.

C'est la même mécanique que Stripe → ton site quand un paiement arrive, ou GitHub → CI quand on push.
