# Mémoire du projet

Mémoire partagée et versionnée : décisions, préférences, points ouverts. À mettre à jour à la fin de chaque tâche significative.

## Point de reprise

- **2026-11-26** — Branche `feature/01-pre-install`. Dépôt en pré-installation : seuls `README.md`, `CLAUDE.md` et la configuration Claude (`.claude/`) existent. Aucun squelette Symfony, aucun `compose.yaml`, aucun test.

## À faire

- [ ] Créer la stack Docker de dev (`compose.yaml` : `nginx`, `php` 8.5-fpm, `database` MariaDB 11.4, `phpmyadmin`, `mailer` Mailpit) — agent `devops`
- [ ] Générer le squelette Symfony 7.4 LTS `--webapp` — agent `dev-symfony`
- [ ] Installer les bundles de la stack (Vich, Liip, EasyAdmin 5, Symfony UX, fixtures + Foundry)
- [ ] Rédiger `docs/devops/docker-setup.md` et `docs/architecture/overview.md` — agent `redacteur-docs`
- [ ] Mettre en place la CI GitHub Actions (préproduction)

## Décisions

- Fixtures de démo : `doctrine/doctrine-fixtures-bundle` + `zenstruck/foundry` (2026-11-26).

## Points ouverts

- Famille d'instruments de musique : **à définir** (`[VOTRE CHOIX]` dans le README) — ne pas inventer.
- Public cible : **à définir** — ne pas inventer.
- Langues étrangères de la future traduction : **à définir**.
