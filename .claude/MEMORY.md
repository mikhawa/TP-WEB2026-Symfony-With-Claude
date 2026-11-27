# Mémoire du projet

Mémoire partagée et versionnée : décisions, préférences, points ouverts. À mettre à jour à la fin de chaque tâche significative.

## Point de reprise

- **2026-11-27** — Branche `feature/03-install-prerequis`. Dépôt toujours en pré-installation : documentation et configuration uniquement (`README.md`, `CLAUDE.md`, `PLAN.md`, `RESUME.md`, `PREREQUIS.md`, `.gitattributes`, `.claude/`, `documentation-dev/`). `PREREQUIS.md` (guide du poste Windows 11 : WSL2 + Ubuntu 24.04, PHP 8.4+ / Composer / Symfony CLI dans WSL, Docker Desktop, Git, SSH GitHub, éditeur, Claude Code) vient d'être rédigé et est lié depuis le `README.md`. Aucun squelette Symfony, aucun `compose.yaml`, aucun test. Prochaine étape : étape 0 (décisions) puis étape 1 (stack Docker, agent `devops`) de `PLAN.md`.

## À faire

- [ ] Créer la stack Docker de dev (`compose.yaml` : `nginx`, `php` 8.5-fpm, `database` MariaDB 11.4, `phpmyadmin`, `mailer` Mailpit) — agent `devops`
- [ ] Générer le squelette Symfony 7.4 LTS `--webapp` — agent `dev-symfony`
- [ ] Installer les bundles de la stack (Vich, Liip, EasyAdmin 5, Symfony UX, fixtures + Foundry)
- [ ] Rédiger `docs/devops/docker-setup.md` et `docs/architecture/overview.md` — agent `redacteur-docs`
- [ ] Mettre en place la CI GitHub Actions (préproduction)

## Décisions

- Fixtures de démo : `doctrine/doctrine-fixtures-bundle` + `zenstruck/foundry` (2026-11-26).
- Poste de dev : PHP 8.4 minimum, Composer et Symfony CLI aussi installés dans WSL pour l'outillage local ; le projet s'exécute toujours dans Docker (commandes via `docker compose exec php`, pas de `symfony serve`) (2026-11-27).

## Points ouverts

- Famille d'instruments de musique : **à définir** (`[VOTRE CHOIX]` dans le README) — ne pas inventer.
- Public cible : **à définir** — ne pas inventer.
- Langues étrangères de la future traduction : **à définir**.
