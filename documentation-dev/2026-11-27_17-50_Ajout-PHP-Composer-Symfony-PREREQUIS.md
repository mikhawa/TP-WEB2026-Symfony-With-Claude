# Ajout de PHP, Composer et Symfony CLI dans PREREQUIS.md

- **Date :** 2026-11-27 17:50
- **Agent :** session principale <Modele: opus>

## Fichier(s) modifié(s)

- `PREREQUIS.md`

## Résumé des changements

- Nouvelle section **4 — Installer PHP, Composer et Symfony CLI** (dans WSL) :
  - PHP **8.4 minimum** via le PPA `ondrej/php` (Ubuntu 24.04 ne fournit que 8.3), extensions `intl`, `mbstring`, `xml`, `curl`, `zip`, `mysql`, `gd`, `opcache` ; option 8.5 pour s'aligner sur le conteneur ; `update-alternatives` ;
  - Composer 2 via l'installeur officiel, déplacé dans `/usr/local/bin/composer` ;
  - Symfony CLI via le dépôt APT officiel, contrôle par `symfony check:requirements` ; avertissement : ne pas utiliser `symfony serve` (port 8000 réservé à Nginx dans Docker).
- Principe clé reformulé : le projet s'exécute toujours dans Docker, l'installation locale sert à l'outillage.
- Sections suivantes renumérotées (5 à 12), sommaire et renvois internes mis à jour.
- Vérification finale, checklist et dépannage complétés (PHP, Composer, Symfony CLI).

## Raison

Demande de l'utilisateur : ajouter PHP 8.4 minimum, Composer et Symfony CLI dans WSL.
