# Étapes du POC (déploiement reproductible)

Date du POC : **27/04/2026**
Environnement : Coolify (serveur LogiSpear, IP `185.158.132.58`)

## 1. GitLab CE sur Coolify

- New Resource → Docker Image
- Image : `gitlab/gitlab-ce`
- Port exposé : `80`
- URL générée par Coolify : `http://gitlab-jgk4c8wo480080wo4wwwgw4w.185.158.132.58.sslip.io`
- Dans le champ Domain de Coolify, **ajouter `:80`** pour que Traefik route correctement vers le port 80 interne du conteneur
- Deploy

Le premier démarrage de GitLab CE prend 2-3 minutes (init de la base, configuration, etc.). Patience.

## 2. Compte admin GitLab

- Aller sur l'URL → écran de login
- Username : `root`
- Mot de passe initial : récupérable en exécutant dans le conteneur GitLab
  ```bash
  docker exec -it <container> grep 'Password:' /etc/gitlab/initial_root_password
  ```
- À la première connexion, GitLab demande de changer le mot de passe

## 3. Projet de test

- Create blank project
- Nom : `pr-agent-test`
- Initialiser avec un README
- Créer un fichier `calculator.py` sur `main` avec du code volontairement pas top :
  ```python
  def add(a, b):
      return a + b

  def divide(a, b):
      return a / b

  def calculate_average(numbers):
      total = 0
      for n in numbers:
          total = total + n
      return total / len(numbers)
  ```
- Créer une branche `feature/improve-calculator` (Code → Branches → New branch)
- Modifier `calculator.py` sur cette branche pour donner matière à review :
  ```python
  def add(a, b):
      return a + b

  def divide(a, b):
      if b == 0:
          return None
      return a / b

  def calculate_average(numbers):
      return sum(numbers) / len(numbers)

  def multiply(a, b):
      result = 0
      for i in range(b):
          result += a
      return result
  ```

## 4. Personal Access Token GitLab

- User Settings (avatar en haut à droite) → Access Tokens → Add new token
- Nom : `pr-agent`
- Scope : `api` (suffit, donne lecture + écriture sur le projet)
- Date d'expiration : 1 an par défaut
- Copier le token qui commence par `glpat-...`

## 5. Clé OpenRouter

- openrouter.ai → Sign in (GitHub OAuth ou autre)
- Keys → Create Key → nommer la clé
- Acheter min $5 de crédit (largement assez pour des centaines de reviews)
- Copier la clé qui commence par `sk-or-v1-...`

## 6. PR-Agent sur Coolify

- New Resource → Docker Image
- Image : `codiumai/pr-agent`
- Tag : `gitlab_webhook` (cette image expose le serveur webhook, pas un autre tag)
- Port exposé : `3000`
- Domain : URL générée par Coolify (sans suffixe `:3000`)
- Variables d'environnement à ajouter :

  | Variable | Valeur |
  |---|---|
  | `CONFIG__GIT_PROVIDER` | `gitlab` |
  | `CONFIG__MODEL` | `openrouter/anthropic/claude-3.5-haiku` |
  | `GITLAB__URL` | URL de notre GitLab CE (ex: `http://gitlab-...sslip.io`) |
  | `GITLAB__PERSONAL_ACCESS_TOKEN` | `glpat-...` (étape 4) |
  | `GITLAB__SHARED_SECRET` | secret au choix, ex: `pr-agent-webhook-poc-1f8a2b3c4d5e6f` |
  | `OPENROUTER__KEY` | `sk-or-v1-...` (étape 5) |

- Save All Environment Variables
- Retour sur General → Deploy
- Vérifier les logs : tu dois voir un truc style `Uvicorn running on http://0.0.0.0:3000`

## 7. Webhook GitLab

- Sur le projet `pr-agent-test` → Settings → Webhooks → Add new webhook
- URL : `<URL de PR-Agent>/webhook`
- Secret token : **identique** à `GITLAB__SHARED_SECRET` mis dans PR-Agent
- Décocher "Enable SSL verification" (on est en HTTP)
- Cocher uniquement les triggers :
  - `Merge request events`
  - `Comments`
- Add webhook

Le bouton "Test" affichera une erreur tant qu'il n'y a pas de MR dans le projet (limitation GitLab côté test, pas un vrai problème).

## 8. Tester

- Code → Merge requests → New merge request
- Source : `feature/improve-calculator` → Target : `main`
- Compare → Create merge request
- Attendre 30-60 secondes, refresh
- PR-Agent doit avoir posté :
  - Une description auto-générée (PR Type, bullet points)
  - Un label `Review effort N/5`
  - Un commentaire `PR Reviewer Guide` avec ses observations (no security concerns, recommended focus areas, etc.)
  - Un commentaire `PR Code Suggestions` avec un tableau de suggestions

## 9. Tester les commandes manuelles

Voir [commandes.md](./commandes.md) pour la liste complète. Les principales :

- `/review` → relance une review
- `/improve` → propose des suggestions de code
- `/ask <question>` → pose une question libre, réponse en analysant le code
- `/describe` → regénère la description de la MR

## Coût constaté pendant le POC

- 7 requêtes (1 review auto + 1 description auto + 4 commandes manuelles + ajustements)
- Total facturé OpenRouter : **0,06€**
- Soit ≈ 1 centime par review

Sur 100 MR/mois → ~1€/mois. Sur 1000 MR/mois → ~10€/mois.
