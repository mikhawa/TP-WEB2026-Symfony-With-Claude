# TP-WEB2026-Symfony-With-Claude

Création d'un site d'e-commerce factice sur les instruments de musique pour les 2 classes de Webdev 2026.

## Projet instruments de musique

Choix de la famille ou des sortes d'instruments :

[VOTRE CHOIX]

## Liste des commandes claudes

- [Complètes](https://github.com/WebDevCF2m2026/Cours-Claude-Code-novembre-2026/blob/main/README.md)
- [Résumé](RESUME.md)

## Prérequis à installer

[Prérequis à installer pour le projet](PREREQUIS.md)

## Plan pour bien débuter

[Plan pour débuter](PLAN.md)

# E-Commerce Sandbox (Symfony 7.4 LTS)

**Objectif** : Présentation d'une famille d'instruments de musique [VOTRE CHOIX] Ce dépôt contient une plateforme e-commerce factice (sandbox / démonstrateur technique) construite avec **Symfony 7.4 LTS**.
L'objectif est de produire un code de qualité production (architecture propre, typage strict, transactions ACID) tout en simulant les processus externes lourds (paiement, expédition, emails).

**Public cible** : Passionnés de musique et [VOTRE CHOIX].

**Langue du projet** : Ce projet se fait entièrement en français : code (commentaires, messages de validation, noms de commits), documentation, et échanges. Par contre, il sera dans un second temps mis en dans des langues étrangères [anglais, Espagnol etc... ou  VOS CHOIX]

**Stack** : Symfony 7.4 LTS --webapp | PHP 8.5 | MariaDB 11.4 | Docker en development | ImportMap | AssetMapper | Symfony UX.

**Stack Docker de dev** : 

En `WSL` sous Windows 11, utilisation d'images Docker

5 services : `nginx`, `php` (8.5-fpm), `database` (MariaDB 11.4), `phpmyadmin`, `mailer` (Mailpit). 
- **Langage :** PHP 8.5+ (`declare(strict_types=1);` obligatoire en haut de chaque fichier PHP).
- **ORM :** Doctrine ORM 3.x (Migrations obligatoires, aucun `doctrine:schema:update --force`).
- **Front-end :** Twig + **AssetMapper** (zéro Node.js / Webpack) + **Symfony UX** (`ux-twig-component`, `ux-live-component`, `ux-turbo`).
- **Gestion des Images :** `VichUploaderBundle` + `LiipImagineBundle` pour la génération de miniatures, stockées dans `public/uploads/products/`.
- **Back-office :** `EasyAdminBundle` 5.

**Environnements** : dev (Docker local) → préproduction (GitHub CI) | La production se fera dans un second temps.

