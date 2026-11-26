# Choix du modèle pour les sous-agents

Règle générale : prendre le modèle le moins coûteux capable de faire la tâche correctement.

| Modèle | Quand l'utiliser | Agents |
|--------|------------------|--------|
| **Haiku** | Tâches mécaniques, bien cadrées, sans décision d'architecture : lancer des commandes et rapporter, rédiger de la doc à partir du code, CRUD EasyAdmin simple. | `verificateur`, `redacteur-docs`, `admin-crud` |
| **Sonnet** | Développement courant : entités, services, contrôleurs, composants Live, migrations, tests, Docker/CI. | `dev-symfony`, `migrations`, `testeur`, `devops` |
| **Opus** | Raisonnement profond : conception, choix structurants, audit de sécurité, bug dont la cause n'est pas évidente. | `architecte`, `auditeur-securite`, `debogueur` |

## Escalade

- Un agent Haiku qui rencontre une décision de conception s'arrête et la signale au lieu de trancher.
- Un agent Sonnet qui bloque après deux tentatives sur un bug passe la main à `debogueur`.
- Toute modification touchant le workflow `order_process`, les montants financiers ou l'authentification passe par `architecte` (conception) puis `auditeur-securite` (relecture).
- Toute modification de code se termine par `verificateur`.
