# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## État du dépôt

Le projet est en phase de pré-installation (branche `feature/01-pre-install`) : aucune application Symfony n'est encore générée, il n'existe ni `composer.json`, ni `compose.yaml`, ni tests. Mettre à jour ce fichier (commandes de build, tests, lint, Docker) dès que le squelette Symfony et la stack Docker seront en place.

## Projet

Site e-commerce **factice** (sandbox / démonstrateur technique) présentant une famille d'instruments de musique, réalisé pour les classes Webdev 2026. La famille d'instruments et le public cible sont encore à définir (`[VOTRE CHOIX]` dans le README).

Objectif : code de qualité production (architecture propre, typage strict, transactions ACID) tout en **simulant** les processus externes lourds (paiement, expédition, emails) — ne jamais intégrer de vrais prestataires.

## Langue

Tout se fait en **français** : code (commentaires, messages de validation), documentation, messages de commit et échanges avec l'utilisateur. Une internationalisation (anglais, espagnol, …) viendra dans un second temps : prévoir les textes d'interface traduisibles.

## Stack imposée

- **Symfony 7.4 LTS** (squelette `--webapp`), **PHP 8.5+**
- `declare(strict_types=1);` obligatoire en tête de chaque fichier PHP
- **Doctrine ORM 3.x** + **MariaDB 11.4** — migrations obligatoires ; ne jamais utiliser `doctrine:schema:update --force`
- Front : **Twig + AssetMapper / ImportMap** (zéro Node.js, zéro Webpack/Encore) + **Symfony UX** (`ux-twig-component`, `ux-live-component`, `ux-turbo`)
- Images : **VichUploaderBundle** + **LiipImagineBundle** (miniatures), fichiers stockés dans `public/uploads/products/`
- Back-office : **EasyAdminBundle 5**

## Environnement de développement

Docker sous WSL (Windows 11), 5 services : `nginx`, `php` (8.5-fpm), `database` (MariaDB 11.4), `phpmyadmin`, `mailer` (Mailpit). Les commandes `php`/`composer`/`bin/console` sont destinées à être exécutées dans le conteneur `php`.

Environnements : dev (Docker local) → préproduction (GitHub CI). La production viendra plus tard.
