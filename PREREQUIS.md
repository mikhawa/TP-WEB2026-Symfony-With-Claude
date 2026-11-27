# Prérequis à installer

[Retour à l'accueil](README.md)

Ce guide prépare un poste **Windows 11** pour travailler sur le projet : WSL2 + Ubuntu, Docker Desktop, Git, GitHub, un éditeur et Claude Code.

> **Principe clé** : PHP, Composer, Symfony, MariaDB, Nginx… **ne s'installent pas sur la machine**. Ils tournent dans les conteneurs Docker du projet (`docker compose exec php ...`). Sur le poste, il ne faut que : **WSL2 + Ubuntu, Docker, Git, un éditeur et Claude Code**.

---

## Sommaire

1. [Vérifier le poste Windows 11](#1--vérifier-le-poste-windows-11)
2. [Installer WSL2 et Ubuntu](#2--installer-wsl2-et-ubuntu)
3. [Configurer Ubuntu](#3--configurer-ubuntu)
4. [Installer Docker Desktop](#4--installer-docker-desktop)
5. [Installer et configurer Git](#5--installer-et-configurer-git)
6. [Connecter GitHub (clé SSH)](#6--connecter-github-clé-ssh)
7. [Cloner le projet](#7--cloner-le-projet)
8. [Éditeur de code](#8--éditeur-de-code)
9. [Installer Claude Code](#9--installer-claude-code)
10. [Vérification finale](#10--vérification-finale)
11. [Dépannage](#11--dépannage)

---

## 1 — Vérifier le poste Windows 11

| Élément | Minimum conseillé |
|---|---|
| Windows | Windows 11 à jour (Windows Update) |
| RAM | 8 Go (16 Go recommandés) |
| Disque libre | 30 Go |
| Virtualisation | Activée dans le BIOS/UEFI |

**Vérifier la virtualisation** : `Ctrl+Maj+Échap` → Gestionnaire des tâches → onglet **Performances** → **Processeur** → « Virtualisation : **Activé** ».

Si elle est désactivée : redémarrer dans le BIOS/UEFI et activer **Intel VT-x** / **AMD-V (SVM)** (le nom varie selon le fabricant).

Installer aussi **Windows Terminal** (présent par défaut sur Windows 11, sinon via le Microsoft Store).

---

## 2 — Installer WSL2 et Ubuntu

Ouvrir **PowerShell en tant qu'administrateur** (clic droit sur le menu Démarrer → « Terminal (administrateur) ») :

```powershell
wsl --install -d Ubuntu-24.04
```

Cette commande active les fonctionnalités Windows nécessaires, installe le noyau WSL2 et la distribution Ubuntu 24.04 LTS.

**Redémarrer le PC** quand c'est demandé. Au redémarrage, Ubuntu s'ouvre et demande :
- un **nom d'utilisateur** Linux (en minuscules, sans espace ni accent) ;
- un **mot de passe** (rien ne s'affiche pendant la saisie, c'est normal). Il servira pour `sudo`.

Vérifier ensuite dans PowerShell :

```powershell
wsl --update
wsl --list --verbose
```

La colonne `VERSION` doit afficher **2** pour Ubuntu. Si ce n'est pas le cas :

```powershell
wsl --set-default-version 2
wsl --set-version Ubuntu-24.04 2
```

### (Optionnel) Limiter la mémoire de WSL

Par défaut WSL peut prendre jusqu'à 50 % de la RAM. Créer le fichier `C:\Users\<VotreNom>\.wslconfig` :

```ini
[wsl2]
memory=6GB
processors=4
swap=2GB
```

Puis appliquer avec `wsl --shutdown` (dans PowerShell) et rouvrir Ubuntu.

---

## 3 — Configurer Ubuntu

Dans le terminal **Ubuntu** :

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget unzip make ca-certificates
```

> ⚠️ **Règle importante** : toujours travailler dans le système de fichiers Linux (`~/...`, soit `/home/<utilisateur>/...`), **jamais dans `/mnt/c/...`**. Sous `/mnt/c`, Docker et Symfony sont très lents et les droits de fichiers posent problème.

Depuis l'Explorateur Windows, les fichiers Ubuntu sont accessibles via `\\wsl$\Ubuntu-24.04\home\<utilisateur>` (ou l'entrée « Linux » dans le panneau de gauche).

---

## 4 — Installer Docker Desktop

1. Télécharger **Docker Desktop pour Windows** : <https://www.docker.com/products/docker-desktop/>
2. Lancer l'installation en laissant cochée l'option **« Use WSL 2 instead of Hyper-V »**.
3. Redémarrer si demandé, puis lancer Docker Desktop (accepter les conditions).
4. Dans **Settings** (roue dentée) :
   - **General** → cocher **« Use the WSL 2 based engine »** ;
   - **Resources → WSL integration** → activer **« Enable integration with my default WSL distro »** et cocher **Ubuntu-24.04** ;
   - cliquer **Apply & restart**.

Vérifier **dans Ubuntu** (Docker Desktop doit être lancé) :

```bash
docker --version
docker compose version
docker run --rm hello-world
```

Le message « Hello from Docker! » confirme que tout fonctionne.

> Il n'est **pas** nécessaire d'installer Docker Engine dans Ubuntu : Docker Desktop fournit les commandes `docker` et `docker compose` à WSL. N'installer qu'**une seule** des deux solutions pour éviter les conflits.

**Ports utilisés par le projet** (doivent être libres sur le poste) : `8000` (site), `8080` (phpMyAdmin), `8025` (Mailpit). Arrêter WAMP, XAMPP, Laragon ou tout autre serveur local qui occuperait ces ports ou le port MySQL `3306`.

---

## 5 — Installer et configurer Git

Git est installé à l'étape 3. Le configurer **dans Ubuntu** :

```bash
git config --global user.name "Prénom Nom"
git config --global user.email "adresse@exemple.com"   # la même que sur GitHub
git config --global init.defaultBranch main
git config --global core.autocrlf input
git config --global pull.rebase false
```

- `core.autocrlf input` : les fins de ligne restent en **LF** (format Linux), en complément du fichier `.gitattributes` du projet.
- Vérifier : `git config --global --list`

> Si **Git pour Windows** est aussi installé (pour un éditeur côté Windows), y régler `git config --global core.autocrlf false` pour ne pas convertir les fichiers en CRLF.

### (Optionnel) GitHub CLI

Pratique pour les pull requests et utilisé par Claude Code :

```bash
sudo apt install -y gh
gh auth login
```

---

## 6 — Connecter GitHub (clé SSH)

1. Créer un compte sur <https://github.com> si nécessaire.
2. Générer une clé SSH **dans Ubuntu** :

   ```bash
   ssh-keygen -t ed25519 -C "adresse@exemple.com"
   # Valider l'emplacement par défaut, choisir une phrase de passe (conseillé)
   cat ~/.ssh/id_ed25519.pub
   ```

3. Copier la clé affichée (ligne commençant par `ssh-ed25519`) dans GitHub → **Settings → SSH and GPG keys → New SSH key**.
4. Tester :

   ```bash
   ssh -T git@github.com
   ```

   Réponse attendue : « Hi <pseudo>! You've successfully authenticated… ».

---

## 7 — Cloner le projet

**Dans Ubuntu**, dans le dossier personnel Linux :

```bash
mkdir -p ~/projets && cd ~/projets
git clone git@github.com:<organisation-ou-pseudo>/TP-WEB2026-Symfony-With-Claude.git
cd TP-WEB2026-Symfony-With-Claude
```

Remplacer `<organisation-ou-pseudo>` par le propriétaire du dépôt (ou de votre fork).

Le démarrage de la stack (`docker compose up -d --build`) est décrit dans le [plan](PLAN.md) (étape 1) une fois le `compose.yaml` créé.

---

## 8 — Éditeur de code

Au choix, les deux fonctionnent avec WSL :

- **PhpStorm** (licence étudiante gratuite via <https://www.jetbrains.com/community/education/>) : ouvrir le projet via `\\wsl$\Ubuntu-24.04\home\<utilisateur>\projets\...` ou **File → Remote Development → WSL**. Plugin conseillé : **Symfony Support**.
- **Visual Studio Code** : installer l'extension **WSL** (Microsoft), puis depuis Ubuntu, dans le dossier du projet :

  ```bash
  code .
  ```

  Extensions conseillées : *PHP Intelephense*, *Twig Language 2*, *YAML*, *Docker*, *EditorConfig*.

---

## 9 — Installer Claude Code

**Dans Ubuntu** :

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Puis, depuis le dossier du projet :

```bash
cd ~/projets/TP-WEB2026-Symfony-With-Claude
claude
```

Au premier lancement, se connecter avec son compte Claude (abonnement) ou une clé API. Claude Code lit automatiquement `CLAUDE.md` et la configuration du dossier `.claude/`.

Vérifier : `claude --version` et `claude doctor`.

Les commandes utiles sont listées dans le [résumé](RESUME.md).

---

## 10 — Vérification finale

Toutes ces commandes se lancent **dans Ubuntu**, Docker Desktop démarré :

```bash
wsl.exe --list --verbose     # Ubuntu en VERSION 2
docker --version             # Docker version ...
docker compose version       # Docker Compose version v2...
docker run --rm hello-world  # Hello from Docker!
git --version                # git version ...
git config --global user.email
ssh -T git@github.com        # Hi <pseudo>!
claude --version             # version de Claude Code
pwd                          # doit commencer par /home/, pas /mnt/c/
```

Checklist :

- [ ] Virtualisation activée
- [ ] WSL2 + Ubuntu 24.04 installés et à jour
- [ ] Docker Desktop installé, intégration WSL activée
- [ ] Git configuré (nom, email, `autocrlf input`)
- [ ] Clé SSH ajoutée sur GitHub
- [ ] Projet cloné dans `/home/<utilisateur>/...`
- [ ] Éditeur connecté à WSL
- [ ] Claude Code installé et connecté
- [ ] Ports `8000`, `8080`, `8025`, `3306` libres

---

## 11 — Dépannage

| Problème | Solution |
|---|---|
| `wsl --install` : erreur de virtualisation (`0x80370102`) | Activer VT-x / AMD-V dans le BIOS, puis relancer. |
| `docker: command not found` dans Ubuntu | Lancer Docker Desktop et vérifier **Resources → WSL integration** (Ubuntu cochée). Fermer/rouvrir le terminal. |
| `permission denied ... docker.sock` | Redémarrer Docker Desktop ; en dernier recours `wsl --shutdown` dans PowerShell. |
| Docker / Symfony très lents | Le projet est sous `/mnt/c/...` : le déplacer dans `~/` (étape 7). |
| `port is already allocated` | Un autre logiciel utilise le port (WAMP, XAMPP, Skype…). L'arrêter. Sous PowerShell : `netstat -ano \| findstr :8000`. |
| Fichiers modifiés partout dans `git status` (fins de ligne) | Vérifier `core.autocrlf input` (Ubuntu) ; ne pas éditer le dépôt avec Git pour Windows réglé en `true`. |
| WSL consomme trop de RAM (`vmmem`) | Créer `.wslconfig` (étape 2) puis `wsl --shutdown`. |
| Horloge décalée dans WSL après une mise en veille | `sudo hwclock -s` ou `wsl --shutdown`. |

---

[Retour à l'accueil](README.md) · [Plan pour débuter](PLAN.md)
