# Configuration Claude et correction de CLAUDE.md

**Date :** 2026-11-26 11:43
**Branche :** `feature/01-pre-install`

Agent: claude-principal <Modele: opus>

## Fichiers modifiés

- `CLAUDE.md`

## Fichiers créés

- `.claude/MEMORY.md`
- `.claude/models.md`
- `.claude/agents/` : `verificateur.md`, `redacteur-docs.md`, `admin-crud.md` (haiku) ; `dev-symfony.md`, `migrations.md`, `testeur.md`, `devops.md` (sonnet) ; `architecte.md`, `auditeur-securite.md`, `debogueur.md` (opus)
- `documentation-dev/` (ce fichier)

## Résumé des changements

- §1 : précise que les §2.1, §3, §4 et §5 décrivent la cible (squelette Symfony et `compose.yaml` pas encore créés).
- §2.2 : ajout de `doctrine/doctrine-fixtures-bundle` + `zenstruck/foundry` dans la stack.
- §3 : libellé « Vérifier le conteneur et le typage » renommé en « Linter le conteneur, Twig et YAML » (`lint:container` ne vérifie pas le typage).
- §2.1 et §9 : les fichiers de `docs/` sont signalés comme « à créer ».
- Numérotation des sections 8 et 9, séparateurs et ponctuation du §7.
- Création de la mémoire projet, des règles de choix de modèle et des 10 sous-agents cités au §6.

## Raison

`CLAUDE.md` référençait des fichiers inexistants (`.claude/MEMORY.md` importé via `@`, `.claude/agents/`, `.claude/models.md`, `documentation-dev/`, `docs/`), ce qui rendait l'import vide et la délégation aux agents impossible. Les choix `[VOTRE CHOIX]` (famille d'instruments, public cible) restent volontairement ouverts.
