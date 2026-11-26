# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# E-Commerce Sandbox (Symfony 7.4 LTS)

## 1. Contexte et Objectif du Projet

Ce dépôt contient une plateforme e-commerce factice (sandbox / démonstrateur technique) construite avec **Symfony 7.4 LTS**.
L'objectif est de produire un code de qualité production (architecture propre, typage strict, transactions ACID) tout en simulant les processus externes lourds (paiement, expédition, emails).

**Langue** : tout en français — commentaires, messages de validation, commits, documentation, échanges. Des langues étrangères pourront être ajoutées dans un second temps au projet.

**État actuel du dépôt :** voir « Point de reprise » et « À faire » dans `.claude/MEMORY.md`. Les §2.1, §3, §4 et §5 décrivent la stack et l'architecture **cibles** (le squelette Symfony et `compose.yaml` ne sont pas encore créés) : vérifier l'existence d'un fichier avant de s'y fier et créer les éléments manquants conformément à ce document.

---

## 2. Stack Technique & Environnement Docker (Mode Dev)
En mode développement, **toutes les briques importantes tournent dans des conteneurs Docker isolés** orchestrés via `compose.yaml` (`docker compose`), avec une séparation stricte entre le serveur HTTP et le runtime PHP :

### 2.1 Services Docker (`compose.yaml`)
Cinq services : `nginx`, `php` (8.5-fpm), `database` (MariaDB 11.4), `phpmyadmin`, `mailer` (Mailpit). Détail : `compose.yaml` et `docs/devops/docker-setup.md` (à créer).

### 2.2 Stack Applicative
- **Langage :** PHP 8.5+ (`declare(strict_types=1);` obligatoire en haut de chaque fichier PHP).
- **ORM :** Doctrine ORM 3.x (Migrations obligatoires, aucun `doctrine:schema:update --force`).
- **Front-end :** Twig + **AssetMapper** (zéro Node.js / Webpack) + **Symfony UX** (`ux-twig-component`, `ux-live-component`, `ux-turbo`).
- **Gestion des Images :** `VichUploaderBundle` + `LiipImagineBundle` pour la génération de miniatures, stockées dans `public/uploads/products/`.
- **Back-office :** `EasyAdminBundle` 5.
- **Données de démo :** `doctrine/doctrine-fixtures-bundle` + `zenstruck/foundry` (factories), en dépendances `--dev`.

---

## 3. Commandes Utiles (Référence Agent — Exécution via le conteneur `php`)
Toutes les commandes CLI (Composer, Symfony Console, PHPUnit) doivent être exécutées **à l'intérieur du conteneur `php`** (`docker compose exec php ...`) :

- **Démarrer la stack Docker :** `docker compose up -d --build`
- **Accès aux interfaces de dev :**
  - Site Symfony (via Nginx) : `http://localhost:8000`
  - phpMyAdmin : `http://localhost:8080`
  - Mailpit (Emails) : `http://localhost:8025`
- **Installer une dépendance Composer :** `docker compose exec php composer require <package>`
- **Linter le conteneur, Twig et YAML :** `docker compose exec php php bin/console lint:container && docker compose exec php php bin/console lint:twig templates/ && docker compose exec php php bin/console lint:yaml config/`
- **Migrations :** `docker compose exec php php bin/console make:migration` puis `docker compose exec php php bin/console doctrine:migrations:migrate -n`
- **Recharger la base de démo (Fixtures Foundry) :** `docker compose exec php php bin/console doctrine:fixtures:load -n`
- **Vérifier le Workflow de commande :** `docker compose exec php php bin/console workflow:dump order_process` (graphe au format DOT ; `debug:config framework workflows` pour la configuration)
- **Lancer les tests :** `docker compose exec php php bin/phpunit`
- **Lancer un seul test :** `docker compose exec php php bin/phpunit tests/Chemin/VersTest.php` ou `docker compose exec php php bin/phpunit --filter nomDeLaMethode`

---

## 4. Architecture & Règles Métier Strictes

### 4.1 Modélisation des Données (Doctrine ORM) & Médias
- **Identifiants :** Utiliser systématiquement des **UUIDv7** (`Symfony\Component\Uid\UuidV7`) comme clés primaires pour les entités `Product`, `Order` et `User` (tri chronologique natif et sécurité des URLs). Privilégier le type Doctrine `uuid`.
- **Montants financiers :** **INTERDICTION ABSOLUE d'utiliser le type `float`** pour les prix, taxes ou totaux. Stocker tous les montants en **centimes sous forme d'entiers (`int`)** (ex. : `1999` pour 19,99 €).
- **Images Produits :** Stocker uniquement le **nom du fichier ou le chemin relatif** en base de données (`imageFilename` en `VARCHAR(255)`), jamais de `BLOB` binaire dans MariaDB. Les fichiers physiques résident dans `public/uploads/products/` et sont servis directement par le conteneur `nginx`.
- **Isolation Historique (Snapshot) :** L'entité `OrderItem` (ligne de commande) doit figurer comme une archive immuable. Elle doit copier le nom du produit (`productName`), son image (`productImage`) et son prix unitaire (`unitPriceInCents`) au moment précis de l'achat.

### 4.2 Panier & Réactivité UI (Symfony UX)
- **Stockage :** Le panier actif est géré par un service dédié `App\Service\CartService` qui manipule la session via `RequestStack::getSession()`. Il ne stocke que les paires `[productUuid => quantity]` pour rester léger.
- **Composants Live :** L'affichage et la modification du panier (incrémentation, suppression, calcul du sous-total) passent exclusivement par un `#[AsLiveComponent]` (`CartLiveComponent` et `AddToCartButtonComponent`). Ne pas écrire de requêtes `fetch()` / AJAX manuelles en JavaScript.

### 4.3 Cycle de Vie de la Commande (`symfony/workflow`)
- **Interdiction des conditions d'état manuelles :** Ne jamais modifier le statut d'une commande via un simple `$order->setStatus('paid')`.
- **Machine à états :** Utiliser le composant Workflow configuré en `state_machine` (nommé `order_process`) avec une `BackedEnum` PHP (`App\Enum\OrderStatus`).
- **États autorisés :** `cart` -> `pending_payment` -> `paid` -> `shipped` (ainsi que `cancelled` et `refunded`).
- **Événements :** La décrémentation des stocks et l'envoi de l'email de confirmation (capturé par le conteneur Mailpit) doivent être déclenchés via un `EventSubscriber` écoutant l'événement `workflow.order_process.completed.pay`.

### 4.4 Paiement Factice (Sandbox Payment)
- **Mode Simulation Interne :** Par défaut, un contrôleur `FakePaymentController` affiche une interface imitant une passerelle bancaire avec 3 boutons d'action :
  1. *Simuler Succès* (déclenche la transition `pay` du workflow).
  2. *Simuler Échec / Fonds insuffisants* (retour au checkout avec message Flash).
  3. *Simuler Abandon* (transition vers `cancelled`).
- **Architecture Extensible :** Isoler la logique derrière une interface `PaymentGatewayInterface` pour permettre de basculer facilement vers Stripe Test Mode (via le composant `Webhook` de Symfony) sans toucher aux contrôleurs.

---

## 5. Conventions de Code & Bonnes Pratiques Symfony 7.4 / PHP 8.5+
1. **Contrôleurs maigres (Thin Controllers) :** Un contrôleur ne contient aucune logique métier ni requête Doctrine complexe. Il délègue aux Services ou Handlers.
2. **Validation automatique des entrées :** Utiliser les attributs Symfony 7 (`#[MapRequestPayload]`, `#[MapQueryString]`, `#[MapEntity]`) associés à des DTOs en `readonly class` validés par `Symfony\Component\Validator\Constraints`.
3. **Injection de dépendances :** Utiliser exclusivement la promotion de propriétés dans le constructeur (`private readonly ...`). Utiliser `#[Autowire]` ou `#[AutowireIterator]` si nécessaire.

---

## 6. Agents spécialisés

Déléguer aux sous-agents de `.claude/agents/` (règles de choix et enchaînements types — nouvelle fonctionnalité, correction de bug, préproduction : `.claude/models.md` ; en cas de doute entre deux niveaux, prendre le plus élevé) :
- Haiku : `verificateur` (tests + lint), `redacteur-docs`, `admin-crud` (EasyAdmin simple)
- Sonnet : `dev-symfony`, `migrations`, `testeur`, `devops`
- Opus : `architecte`, `auditeur-securite`, `debogueur`

Toute modification de code se termine par un passage de `verificateur`.

---

## 7. Commits et documentation

Claude ne peut pas faire de commits lui-même, ni créer de branche, mais il doit générer des messages de documentation à chaque action dans le dossier `documentation-dev/`. Toujours utiliser le format suivant :
- Format : YYYY-MM-DD_HH-MM_Description-courte.md
- L'horodatage (obtenu via `date +%Y-%m-%d_%H-%M`) sert d'ordre chronologique : aucune numérotation. Si un fichier porte déjà le même horodatage, regrouper les actions dans ce fichier plutôt que d'en créer un second.
- Ajouter la liste d'agents et leurs actions : `Agent: <nom de l'agent <Modele: <haiku|sonnet|opus> >`.
- Contenu minimal : date, fichier(s) modifié(s), résumé des changements, raison


---

## 8. Mémoire du projet

Mémoire partagée et versionnée (décisions, préférences, points ouverts) — à consulter et à tenir à jour :

@.claude/MEMORY.md

---

## 9. Documentation de référence

À lire seulement quand la tâche le demande. **Ces fichiers ne sont pas encore rédigés** : les créer (agent `redacteur-docs`) au fur et à mesure que les éléments correspondants existent.
- Architecture : `docs/architecture/overview.md`
- Schéma BDD : `docs/architecture/database-schema.md`
- Décisions : `docs/architecture/decisions/`
- Docker : `docs/devops/docker-setup.md`
- GitHub Actions : `docs/devops/github-actions.md`
- Déploiement VPS : `docs/devops/vps-preprod.md`