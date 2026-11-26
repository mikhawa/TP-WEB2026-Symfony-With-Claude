# Choix du modèle et de l'agent

Règle générale : prendre le modèle **le moins coûteux capable de faire la tâche correctement**. En cas de doute entre deux niveaux, prendre le plus élevé.

## Haiku — tâches mécaniques, faible risque

| Agent | Quand l'utiliser |
|-------|------------------|
| `verificateur` | Après **toute** modification de code : tests PHPUnit + lints (container, Twig, YAML). Ne corrige rien, rapporte. |
| `redacteur-docs` | Rédiger ou mettre à jour la documentation (`docs/`, `documentation-dev/`, `README.md`). |
| `admin-crud` | CRUD EasyAdmin simple sur une entité existante (champs, filtres, menu), sans logique métier. |

## Sonnet — développement courant

| Agent | Quand l'utiliser |
|-------|------------------|
| `dev-symfony` | Implémenter une fonctionnalité : entités, services, contrôleurs, Live Components, Twig, workflow. |
| `migrations` | Créer, relire ou réparer une migration Doctrine ; vérifier la cohérence schéma ↔ entités. |
| `testeur` | Écrire des tests PHPUnit et des factories/stories Foundry. |
| `devops` | Docker (`compose.yaml`, `docker/`), Nginx, GitHub Actions, déploiement VPS. |

## Opus — raisonnement profond, décisions structurantes

| Agent | Quand l'utiliser |
|-------|------------------|
| `architecte` | Concevoir une fonctionnalité transverse, trancher un choix d'architecture, rédiger un ADR. |
| `auditeur-securite` | Revue de sécurité (authentification, paiement, uploads, injections, CSRF, contrôle d'accès). |
| `debogueur` | Bug non trivial, cause inconnue, comportement intermittent, échec que `dev-symfony` n'a pas résolu. |

## Enchaînements types

- Nouvelle fonctionnalité : `architecte` (si structurante) → `dev-symfony` → `migrations` (si schéma modifié) → `testeur` → `verificateur`.
- Correction de bug : `debogueur` → `testeur` (test de non-régression) → `verificateur`.
- Avant une mise en préproduction : `auditeur-securite` → `devops` → `verificateur`.

Les agents ne font **ni commit ni branche**. Ils terminent par un compte rendu (fichiers modifiés, résumé, raison) que la session principale reporte dans `documentation-dev/` au format `Agent: <nom> <Modele: <haiku|sonnet|opus>>`.
