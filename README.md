# SkillHub - Plateforme e-learning collaborative

SkillHub met en relation des **formateurs** et des **apprenants** autour de formations en ligne.

Ce document est un guide d'onboarding detaille pour les coequipiers du projet (frontend, backend, devops).

## Sommaire

- [1. Contexte et objectifs](#1-contexte-et-objectifs)
- [2. Vision produit](#2-vision-produit)
- [3. Architecture globale](#3-architecture-globale)
- [4. Stack technique](#4-stack-technique)
- [5. Structure du repository](#5-structure-du-repository)
- [6. Roles et responsabilites](#6-roles-et-responsabilites)
- [7. Prerequis local](#7-prerequis-local)
- [8. Installation complete (pas a pas)](#8-installation-complete-pas-a-pas)
- [9. Lancement en local](#9-lancement-en-local)
- [10. Verification fonctionnelle rapide](#10-verification-fonctionnelle-rapide)
- [11. Workflow equipe (Git)](#11-workflow-equipe-git)
- [12. Qualite, tests et definition of done](#12-qualite-tests-et-definition-of-done)
- [13. Livrables Bloc 03 (rappel)](#13-livrables-bloc-03-rappel)
- [14. Depannage avance](#14-depannage-avance)
- [15. Documentation detaillee](#15-documentation-detaillee)

## 1. Contexte et objectifs

Le projet SkillHub est construit en plusieurs phases. La phase actuelle vise:

- une application fonctionnelle frontend + backend,
- une securisation de l'authentification via JWT,
- une industrialisation progressive (Docker, CI/CD, architecture cloud),
- une documentation claire pour un travail equipe efficace.

Ce README sert de base commune pour reduire les blocages onboarding et harmoniser les pratiques.

## 2. Vision produit

SkillHub doit permettre:

- aux formateurs de publier et gerer leurs contenus,
- aux apprenants de decouvrir, suivre et progresser,
- a l'equipe technique de maintenir la plateforme de facon fiable et evolutive.

Fonctions coeur:

- catalogue de formations,
- details d'une formation + modules,
- gestion des modules par le formateur (ajout, modification, suppression, ordre),
- inscription/desinscription apprenant,
- dashboard formateur,
- dashboard apprenant,
- validation client renforcee sur les formulaires critiques (auth, inscription, formations, modules),
- compteur de vues backend avec exclusion du proprietaire + cooldown anti-refresh,
- journalisation d'activites (MongoDB) pour audit technique.

Regles d'inscription actuelles:

- `prenom`, `nom` et `telephone` (champ `contact`) sont obligatoires pour les nouveaux comptes.
- L'email reste l'identifiant principal de connexion et de contact.
- Les comptes deja existants restent compatibles (colonnes ajoutees en base sans forcer de mise a jour immediate).

## 3. Architecture globale

```text
Frontend React (Vite)  <----HTTP/JSON---->  Backend Laravel API  <---->  MySQL
       |                                            |
       |                                            +---- JWT Auth + controle des roles
       |
       +--------------------------------------------+---- MongoDB (activity logs)
```

### Flux principal

1. L'utilisateur interagit avec le frontend React.
2. Le frontend appelle l'API Laravel (`/api/*`).
3. Laravel verifie JWT + role (`formateur` ou `apprenant`).
4. Les donnees metier sont lues/ecrites dans MySQL.
5. Les evenements metier importants sont logges dans MongoDB (`activity_logs`).

Regles metier recentes:

- Les formulaires frontend appliquent des regles de validation locales (format, longueur, caracteres autorises) avant l'appel API.
- Sur `GET /api/formations/{formation}`, le backend n'incremente pas les vues pour le formateur proprietaire et applique un cooldown de 15 minutes par visiteur.

## 4. Stack technique

| Cote | Technologie |
|---|---|
| Frontend | React 19, Vite, React Router, React Bootstrap |
| Backend | Laravel 13, PHP 8.3 |
| Auth | JWT (`php-open-source-saver/jwt-auth`) |
| DB principale | MySQL |
| DB logs | MongoDB |
| Tests backend | PHPUnit |

## 5. Structure du repository

```text
skillhub_group/
├── frontend/                  # Application React
│   ├── src/components/        # UI partages
│   ├── src/pages/             # Pages principales
│   ├── src/services/          # Client API + services metier
│   └── README.md              # Guide frontend detaille
├── backend/                   # API Laravel
│   ├── app/Http/Controllers/Api/
│   ├── app/Services/
│   ├── routes/api.php
│   ├── docs/openapi.yaml
│   └── README.md              # Guide backend detaille
└── README.md                  # Guide global (ce fichier)
```

## 6. Roles et responsabilites

Cette section distingue:

- les roles applicatifs (dans le produit),
- les roles equipe Bloc 03 (dans l'organisation projet).

### 6.1 Roles applicatifs

| Role | Ce que le role peut faire | Pages / endpoints cle |
|---|---|---|
| `formateur` | Creer, modifier, supprimer ses formations; gerer les modules de chaque formation (ajout, edition, suppression, ordre); suivre son dashboard | `/dashboard/formateur`, `POST/PUT/DELETE /api/formations/*`, `POST /api/formations/{formation}/modules`, `PUT/DELETE /api/modules/{module}` |
| `apprenant` | Parcourir catalogue, s'inscrire/desinscrire, suivre sa progression | `/formations`, `/dashboard/apprenant`, `POST/DELETE /api/formations/{id}/inscription`, `GET /api/apprenant/formations` |

### 6.2 Roles equipe Bloc 03

| Role | Mission principale | Livrables pilotes |
|---|---|---|
| Cloud Architect | Definir l'architecture cloud cible, comparer les options, produire la recommandation | Rapport d'audit, C4 (C1/C2), budget N1/N2 |
| DevOps Engineer | Industrialiser l'execution et le deploiement | Dockerfiles, `docker-compose.yml`, pipeline CI/CD |
| Tech Lead | Coherence technique, qualite de livraison, gouvernance Git | `CONTRIBUTING.md`, strategy branches/PR, controle qualite |

### 6.3 Regles de collaboration attendues

- Branches: `main`, `develop`, `feature/*`, `fix/*`.
- Pas de commit direct sur `main`.
- PR obligatoires avec description claire + verification locale.
- Conventional Commits (`feat:`, `fix:`, `docs:`, `ci:`, `docker:`, `chore:`).
- Historique Git doit montrer une contribution reguliere des 3 membres.

## 7. Prerequis local

### Outils obligatoires

- Node.js 18+
- npm 9+
- PHP 8.3+
- Composer 2+
- MySQL 8+ (ou MariaDB)
- MongoDB Community Server

### Outils recommandes

- MongoDB Compass (visualisation)
- mongosh (diagnostic)
- GitHub Desktop ou CLI Git

### Notes importantes

- Verifier que MySQL et MongoDB tournent avant de lancer l'API.
- Sur Windows, lancer `mongod` en service est le plus stable.

## 8. Installation complete (pas a pas)

## 8.1 Cloner et se placer a la racine

```bash
git clone <url-du-repo>
cd skillhub_group
```

### 8.2 Backend

```bash
cd backend
composer install
cp .env.example .env
php artisan key:generate
php artisan jwt:secret
```

Configurer `backend/.env`:

```dotenv
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=skillhub
DB_USERNAME=root
DB_PASSWORD=

MONGODB_URI=mongodb://127.0.0.1:27017
MONGODB_HOST=127.0.0.1
MONGODB_PORT=27017
MONGODB_DATABASE=skillhub_logs
MONGODB_USERNAME=
MONGODB_PASSWORD=
MONGODB_AUTH_DATABASE=admin

AUTH_GUARD=api
```

Puis migrer:

```bash
php artisan migrate
```

### 8.3 Frontend

```bash
cd ../frontend
npm install
```

Configurer `frontend/.env`:

```dotenv
VITE_API_BASE_URL=http://localhost:8000
```

## 9. Lancement en local

### Terminal 1 - Backend

```bash
cd backend
php artisan optimize:clear
php artisan serve --host=127.0.0.1 --port=8000
```

### Terminal 2 - Frontend

```bash
cd frontend
npm run dev
```

URLs:

- Frontend: `http://localhost:5173`
- Backend API: `http://127.0.0.1:8000`

## 10. Verification fonctionnelle rapide

Checklist de smoke test:

1. Ouvrir `http://localhost:5173`.
2. Creer un compte `formateur` avec `prenom`, `nom`, `telephone`, `email`, `mot de passe`.
3. Creer une formation depuis le dashboard formateur.
4. Ouvrir le bouton `Modules` de cette formation et ajouter un module.
5. Modifier un module, puis verifier l'ordre et le contenu affiches sur `/formation/:id`.
6. Supprimer un module (en respectant la regle backend de minimum 3 modules).
7. Se deconnecter, creer un compte `apprenant` avec les memes champs obligatoires.
8. S'inscrire a la formation.
9. Verifier le dashboard apprenant.
10. Verifier que `GET /api/formations` repond 200.

Option verification logs MongoDB:

```bash
mongosh "mongodb://127.0.0.1:27017/skillhub_logs" --eval "db.activity_logs.countDocuments()"
```

## 11. Workflow equipe (Git)

Workflow recommande:

1. Partir de `develop` a jour.
2. Creer une branche `feature/nom-court`.
3. Commits petits et atomiques.
4. Push + Pull Request vers `develop`.
5. Review + corrections.
6. Merge quand la PR est validee.

Template message de commit:

```text
feat(frontend): ajouter filtre categorie dans le catalogue
fix(api): corriger verification du role formateur
docs(readme): clarifier setup mongodb windows
```

## 12. Qualite, tests et definition of done

### 12.1 Verification minimale avant PR

Backend:

```bash
cd backend
composer run test
php artisan optimize:clear
```

Frontend:

```bash
cd frontend
npm run build
```

### 12.2 Definition of done (DoD)

Une tache est consideree terminee si:

- le code compile/execute localement,
- les cas principaux sont verifies,
- la doc impactee est mise a jour,
- la PR est lisible et reviewable,
- aucun secret n'est committe.

### 12.3 Rappels qualite frontend/backend

- Frontend: les erreurs de saisie doivent etre captees cote client avant soumission API (auth, creation/edition formations, modules).
- Backend: les regles metier sensibles restent verifiees cote serveur (roles, proprietaire de formation, logique de vues).

## 13. Livrables Bloc 03 (rappel)

Pour rester aligne avec le CDC:

- rapport cloud (analyse, comparaison, recommandation, budget),
- diagrammes C4 (C1 et C2 obligatoires),
- conteneurisation complete (`Dockerfile`, `docker-compose.yml`),
- CI/CD (lint, tests, build, push image tag SHA),
- `CONTRIBUTING.md` et workflow Git propre.

## 14. Depannage avance

### 14.1 Frontend: `Failed to fetch`

- Verifier `VITE_API_BASE_URL`.
- Verifier que l'API tourne sur `127.0.0.1:8000`.
- Verifier CORS backend.

```bash
cd backend
php artisan optimize:clear
```

### 14.2 API: erreurs 500 sur `/api/login` ou `/api/formations`

Ca arrive souvent si la DB configuree n'est pas disponible.

- Verifier `DB_CONNECTION` et credentials dans `backend/.env`.
- Verifier que MySQL est demarre.
- Verifier migrations.

```bash
cd backend
php artisan migrate:status
```

### 14.3 MongoDB: collection vide dans Compass

- Verifier service MongoDB actif.
- Verifier `MONGODB_DATABASE=skillhub_logs`.
- Vider cache Laravel + redemarrer serveur.
- Declencher un event (`/api/register`), puis refresh Compass.

```bash
cd backend
php artisan optimize:clear
```

### 14.4 JWT: 401 Unauthorized

- Token absent/expire.
- `JWT_SECRET` manquant.
- Header `Authorization: Bearer <token>` absent.

Regenerer le secret si necessaire:

```bash
cd backend
php artisan jwt:secret
```

### 14.5 Composer: extension `ext-sodium` manquante

Si `composer install` signale `ext-sodium` manquante:

- activer l'extension sodium dans `php.ini`,
- puis relancer `composer install`.

## 15. Documentation detaillee

- Guide backend: `backend/README.md`
- Guide frontend: `frontend/README.md`
- OpenAPI: `backend/docs/openapi.yaml`
- CDC Bloc 03: `CDC_Bloc03_SkillHub.pdf`

---

Si vous rejoignez le projet: commencez par ce README, puis passez a votre scope (`frontend` ou `backend`) pour les details techniques.
