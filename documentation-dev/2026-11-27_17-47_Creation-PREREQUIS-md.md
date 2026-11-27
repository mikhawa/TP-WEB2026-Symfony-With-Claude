# Création du guide des prérequis PREREQUIS.md

- **Date :** 2026-11-27 17:47
- **Agent :** session principale <Modele: opus>

## Fichier(s) modifié(s)

- `PREREQUIS.md` (création, à la racine)

## Résumé des changements

Guide d'installation du poste de développement sous Windows 11 en 11 sections : vérification du poste (virtualisation, RAM), installation de WSL2 + Ubuntu 24.04 (`wsl --install`, `.wslconfig` optionnel), configuration d'Ubuntu, Docker Desktop avec backend WSL2 et intégration Ubuntu, configuration de Git (`core.autocrlf input`, cohérent avec `.gitattributes`), GitHub CLI, clé SSH GitHub, clonage du projet dans `/home` (pas `/mnt/c`), éditeur (PhpStorm ou VS Code + WSL), installation de Claude Code, vérification finale avec checklist et tableau de dépannage.

Rappel central : PHP, Composer et MariaDB ne s'installent pas sur le poste, ils tournent dans les conteneurs Docker.

## Raison

Demande de l'utilisateur. Le `README.md` pointait vers `PREREQUIS.md` (section « Prérequis à installer »), qui n'existait pas.
