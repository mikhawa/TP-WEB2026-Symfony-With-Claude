# Plan pour bien débuter

[Retour à l'accueil](README.md)

Feuille de route conseillée par Claude pour faire avancer le projet, de la stack Docker jusqu'à la préproduction.
Chaque étape indique l'agent à utiliser (voir `.claude/models.md`) et se termine par un passage de `verificateur`.

> **Méthode conseillée pour chaque étape**
> 1. Créer une branche dédiée (ex. : `feature/03-docker`) — Claude ne crée ni branche ni commit.
> 2. `/clear` avant de commencer : une étape = une conversation propre.
> 3. Pour une étape structurante, passer en mode plan (`Shift+Tab`) ou demander l'agent `architecte` avant de coder.
> 4. Terminer par `verificateur`, relire le `git diff`, puis committer soi-même.
> 5. Vérifier qu'une note a bien été ajoutée dans `documentation-dev/` et que `.claude/MEMORY.md` est à jour.

---

## Étape 0 — Décisions et préalables

Ces choix bloquent la suite du projet (modèle de données, fixtures, textes) : **à trancher avant l'étape 4**.

- [ ] Choisir la **famille d'instruments** (remplacer `[VOTRE CHOIX]` dans le `README.md`).
- [ ] Définir le **public cible**.
- [ ] Lister les **langues étrangères** de la future traduction (peut attendre l'étape 13).
- [ ] Reporter ces décisions dans `.claude/MEMORY.md` (section « Décisions »).
- [x] Ajouter un `.gitattributes` (fins de ligne LF).
- [x] Mettre à jour le « Point de reprise » de `.claude/MEMORY.md` (fait le 2026-11-27, branche `feature/03-install-prerequis`).

## Étape 1 — Stack Docker de dev

**Agent :** `devops` (Sonnet)

- [ ] `compose.yaml` avec les 5 services : `nginx`, `php` (8.5-fpm), `database` (MariaDB 11.4), `phpmyadmin`, `mailer` (Mailpit).
- [ ] Dossier `docker/` : `Dockerfile` PHP (extensions `intl`, `pdo_mysql`, `gd`/`imagick` pour Liip, `opcache`, Composer), configuration Nginx.
- [ ] Volumes persistants pour la base, healthcheck MariaDB.
- [ ] Ports : site `8000`, phpMyAdmin `8080`, Mailpit `8025`.
- [ ] Attention WSL : travailler dans le système de fichiers Linux (`/home/...`), pas dans `/mnt/c/...` (performances et droits).
- [ ] Vérification : `docker compose up -d --build` puis `docker compose ps` (tous les services `healthy` / `running`).

## Étape 2 — Squelette Symfony 7.4 LTS

**Agent :** `dev-symfony` (Sonnet)

- [ ] Générer le squelette `--webapp` **dans le conteneur `php`**.
- [ ] Configurer `.env` / `.env.local` : `DATABASE_URL` (MariaDB 11.4), `MAILER_DSN` (Mailpit).
- [ ] Vérifier que `http://localhost:8000` affiche la page d'accueil Symfony.
- [ ] Ne jamais committer `.env.local` ni de secrets.

## Étape 3 — Installation des bundles

**Agent :** `dev-symfony` (Sonnet)

- [ ] `vich/uploader-bundle`, `liip/imagine-bundle`
- [ ] `easycorp/easyadmin-bundle` (v5)
- [ ] `symfony/ux-twig-component`, `symfony/ux-live-component`, `symfony/ux-turbo`
- [ ] `symfony/workflow`
- [ ] En `--dev` : `doctrine/doctrine-fixtures-bundle`, `zenstruck/foundry`
- [ ] Lints : `lint:container`, `lint:twig`, `lint:yaml` (via `verificateur`).

## Étape 4 — Modèle de données

**Agents :** `architecte` (Opus) → `dev-symfony` → `migrations` → `verificateur`

- [ ] Faire concevoir le schéma par `architecte` (et le documenter dans `docs/architecture/database-schema.md`).
- [ ] Entités prévues : `User`, `Product` (et éventuellement une catégorie / un type d'instrument, à décider), `Order`, `OrderItem`.
- [ ] Règles à respecter (CLAUDE.md §4.1) :
  - UUIDv7 pour `Product`, `Order`, `User` ;
  - montants en **centimes (`int`)**, jamais de `float` ;
  - `imageFilename` en `VARCHAR(255)`, jamais de BLOB ;
  - `OrderItem` = snapshot immuable (`productName`, `productImage`, `unitPriceInCents`).
- [ ] Enum `App\Enum\OrderStatus` (BackedEnum).
- [ ] Migrations uniquement (`make:migration` + `doctrine:migrations:migrate`), jamais de `schema:update --force`.

## Étape 5 — Données de démo

**Agent :** `testeur` (Sonnet) — propriétaire des factories / stories Foundry

- [ ] Factories Foundry pour chaque entité, avec des données réalistes pour la famille d'instruments choisie.
- [ ] Story de démo + `doctrine:fixtures:load -n`.
- [ ] Quelques images de démo dans `public/uploads/products/`.

## Étape 6 — Catalogue (front public)

**Agent :** `dev-symfony` (Sonnet)

- [ ] Layout Twig de base (AssetMapper, zéro Node.js).
- [ ] Liste des produits, page détail (URL avec UUID), miniatures LiipImagine.
- [ ] Contrôleurs maigres : la logique va dans des services / repositories.

## Étape 7 — Authentification

**Agents :** `dev-symfony` → `auditeur-securite` (Opus)

- [ ] Inscription, connexion, déconnexion, rôles `ROLE_USER` / `ROLE_ADMIN`.
- [ ] Protection CSRF, hachage des mots de passe, contrôle d'accès.

## Étape 8 — Panier (Symfony UX Live)

**Agents :** `architecte` (si besoin) → `dev-symfony` → `testeur`

- [ ] `App\Service\CartService` (session, paires `[productUuid => quantity]`).
- [ ] `AddToCartButtonComponent` et `CartLiveComponent` (`#[AsLiveComponent]`), **aucun `fetch()` manuel**.
- [ ] Tests unitaires du `CartService`.

## Étape 9 — Commande et workflow `order_process`

**Agents :** `architecte` (Opus) → `dev-symfony` → `testeur`

- [ ] `state_machine` `order_process` : `cart` → `pending_payment` → `paid` → `shipped` (+ `cancelled`, `refunded`).
- [ ] Création de la commande depuis le panier, avec snapshot des `OrderItem`, dans une transaction.
- [ ] Interdiction de `$order->setStatus(...)` direct : uniquement des transitions.
- [ ] Vérifier avec `workflow:dump order_process`.

## Étape 10 — Paiement factice

**Agents :** `dev-symfony` → `testeur` → `auditeur-securite`

- [ ] `PaymentGatewayInterface` + implémentation factice.
- [ ] `FakePaymentController` avec 3 boutons : Succès (`pay`), Échec (flash + retour checkout), Abandon (`cancelled`).
- [ ] `EventSubscriber` sur `workflow.order_process.completed.pay` : décrémentation du stock + email de confirmation (visible dans Mailpit).
- [ ] Tests fonctionnels des 3 scénarios.

## Étape 11 — Back-office EasyAdmin

**Agent :** `admin-crud` (Haiku), `dev-symfony` si logique métier

- [ ] Dashboard + CRUD `Product` (upload d'image via Vich), `User`, `Order` (lecture seule + transitions de workflow).
- [ ] Accès réservé à `ROLE_ADMIN`.

## Étape 12 — Qualité, sécurité et documentation

**Agents :** `testeur`, `auditeur-securite`, `redacteur-docs`

- [ ] Couverture de tests sur les parties critiques (panier, commande, paiement, stock).
- [ ] Audit de sécurité complet (uploads, CSRF, contrôle d'accès, secrets).
- [ ] Rédiger `docs/architecture/overview.md`, `docs/devops/docker-setup.md`, ADR dans `docs/architecture/decisions/`.
- [ ] Optionnel : ajouter PHPStan et PHP-CS-Fixer (à décider et noter dans `MEMORY.md`).

## Étape 13 — CI et préproduction

**Agents :** `auditeur-securite` → `devops` → `verificateur`

- [ ] GitHub Actions : installation, lints, migrations sur une base de test, PHPUnit à chaque push / pull request.
- [ ] Déploiement sur VPS de préproduction (`docs/devops/vps-preprod.md`).

## Plus tard

- [ ] Traduction (Symfony Translation) dans les langues choisies à l'étape 0.
- [ ] Passerelle Stripe en mode test (composant `Webhook`), derrière `PaymentGatewayInterface`.
- [ ] Expédition simulée (transition `shipped`) et remboursements (`refunded`).
- [ ] Production.

---

## Conseils d'utilisation de Claude Code

- **Une étape = une branche = une conversation.** `/clear` entre deux tâches pour économiser le contexte.
- **Planifier avant de coder** pour les étapes 4, 8, 9 et 10 : mode plan ou agent `architecte`.
- **Toujours relire le `git diff`** avant de committer : Claude ne committe jamais.
- **Toutes les commandes passent par le conteneur** : `docker compose exec php ...`.
- **Ne pas inventer** ce qui n'est pas décidé (famille d'instruments, public cible) : le noter dans `MEMORY.md` dès que c'est tranché.
- **`/code-review`** avant chaque pull request, **`/context`** si une session devient lente.
