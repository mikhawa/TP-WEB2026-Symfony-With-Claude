# Prérequis à installer

[Retour à l'accueil](README.md)

## Requis préparés en classe

C'est requis seront vu ensemble en classe

### Guide de pré-installation

Ce guide prépare un poste **Windows 11** pour travailler sur le projet : WSL2 + Ubuntu, PHP, Composer, Symfony CLI, Docker Desktop, Git, GitHub, un éditeur et Claude Code.

> **Principe clé** : le projet **s'exécute** dans les conteneurs Docker (`docker compose exec php ...`) : MariaDB, Nginx, phpMyAdmin et Mailpit ne s'installent pas sur la machine. PHP, Composer et Symfony CLI sont **aussi** installés dans WSL pour l'outillage local (éditeur, autocomplétion, `symfony check:requirements`, commandes rapides), mais les commandes du projet restent lancées dans le conteneur `php`.

---

## Sommaire

1. [Vérifier le poste Windows 11](#1--vérifier-le-poste-windows-11)
2. [Installer WSL2 et Ubuntu](#2--installer-wsl2-et-ubuntu)
3. [Configurer Ubuntu](#3--configurer-ubuntu)
4. [Installer PHP, Composer et Symfony CLI](#4--installer-php-composer-et-symfony-cli)
5. [Installer Docker Desktop](#5--installer-docker-desktop)
6. [Installer et configurer Git](#6--installer-et-configurer-git)
7. [Connecter GitHub (clé SSH)](#7--connecter-github-clé-ssh)
8. [Cloner le projet](#8--cloner-le-projet)
9. [Éditeur de code](#9--éditeur-de-code)
10. [Installer Claude Code](#10--installer-claude-code)
11. [Vérification finale](#11--vérification-finale)
12. [Dépannage](#12--dépannage)

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

## 4 — Installer PHP, Composer et Symfony CLI

### 4.1 PHP 8.4 minimum

Ubuntu 24.04 ne fournit que PHP 8.3 : on ajoute le dépôt **ondrej/php** (référence pour les versions récentes de PHP sur Ubuntu).

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:ondrej/php
sudo apt update
```

Installer PHP **8.4** (minimum exigé) et les extensions utiles à Symfony :

```bash
sudo apt install -y php8.4-cli php8.4-common php8.4-intl php8.4-mbstring \
  php8.4-xml php8.4-curl php8.4-zip php8.4-mysql php8.4-gd php8.4-opcache
```

> 💡 Le conteneur Docker du projet utilise **PHP 8.5**. Pour avoir la même version en local, remplacer `8.4` par `8.5` dans la commande ci-dessus (si le paquet est disponible : `apt-cache policy php8.5-cli`).

Si plusieurs versions sont installées, choisir celle par défaut :

```bash
sudo update-alternatives --config php
```

Vérifier :

```bash
php -v        # PHP 8.4.x (ou 8.5.x)
php -m        # liste des extensions (intl, mbstring, pdo_mysql, gd…)
```

### 4.2 Composer

Installation officielle (<https://getcomposer.org/download/>) :

```bash
cd ~
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

> Pour vérifier l'intégrité de l'installeur (somme SHA-384), suivre la commande à jour donnée sur la page officielle ci-dessus.

Vérifier :

```bash
composer --version   # Composer version 2.x
```

### 4.3 Symfony CLI

Installation via le dépôt APT officiel de Symfony :

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
sudo apt install -y symfony-cli
```

Vérifier :

```bash
symfony version
symfony check:requirements   # doit se terminer par « [OK] Your system is ready to run Symfony projects »
```

> ⚠️ Pour ce projet, le serveur web est **Nginx dans Docker** (`http://localhost:8000`) : ne pas utiliser `symfony serve`, qui occuperait aussi le port 8000.

---

## 5 — Installer Docker Desktop

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

## 6 — Installer et configurer Git

Inutile si vous avez accès à `Git` via le chemin `Windows 11` vers `WSL2` (`\\wsl$\Ubuntu-24.04\home\<utilisateur>`)

Si vous souhaitez l'utiliser via Ubuntu : 

Git est installé à la section 3. Le configurer **dans Ubuntu** :

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


---

## 7 — Connecter GitHub (clé SSH)

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

## 8 — Cloner le projet

**Dans Ubuntu**, dans le dossier personnel Linux :

```bash
mkdir -p ~/projets && cd ~/projets
git clone git@github.com:<organisation-ou-pseudo>/TP-WEB2026-Symfony-With-Claude.git
cd TP-WEB2026-Symfony-With-Claude
```

**Sans Windows, via le chemin WSL**

```bash
git clone git@github.com:<organisation-ou-pseudo>/TP-WEB2026-Symfony-With-Claude.git
cd TP-WEB2026-Symfony-With-Claude
git status
# rajoutez l'exception git
```

Remplacer `<organisation-ou-pseudo>` par le propriétaire du dépôt (ou de votre fork).

Le démarrage de la stack (`docker compose up -d --build`) est décrit dans le [plan](PLAN.md) (étape 1 du plan) une fois le `compose.yaml` créé.

---

## 9 — Éditeur de code

Au choix, les deux fonctionnent avec WSL :

- **PhpStorm** (licence étudiante gratuite via <https://www.jetbrains.com/community/education/>) : ouvrir le projet via `\\wsl$\Ubuntu-24.04\home\<utilisateur>\projets\...` ou **File → Remote Development → WSL**. Plugin conseillé : **Symfony Support**.
- **Visual Studio Code** : installer l'extension **WSL** (Microsoft), puis depuis Ubuntu, dans le dossier du projet :

  ```bash
  code .
  ```

  Extensions conseillées : *PHP Intelephense*, *Twig Language 2*, *YAML*, *Docker*, *EditorConfig*.

---

## 10 — Installer Claude Code

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

## 11 — Vérification finale

Toutes ces commandes se lancent **dans Ubuntu**, Docker Desktop démarré :

```bash
wsl.exe --list --verbose     # Ubuntu en VERSION 2
docker --version             # Docker version ...
docker compose version       # Docker Compose version v2...
docker run --rm hello-world  # Hello from Docker!
php -v                       # PHP 8.4.x minimum
composer --version           # Composer version 2.x
symfony check:requirements   # [OK] Your system is ready...
git --version                # git version ...
git config --global user.email
ssh -T git@github.com        # Hi <pseudo>!
claude --version             # version de Claude Code
pwd                          # doit commencer par /home/, pas /mnt/c/
```

Checklist :

- [ ] Virtualisation activée
- [ ] WSL2 + Ubuntu 24.04 installés et à jour
- [ ] PHP 8.4+ (extensions intl, mbstring, xml, pdo_mysql…), Composer 2 et Symfony CLI installés dans WSL
- [ ] Docker Desktop installé, intégration WSL activée
- [ ] Git configuré (nom, email, `autocrlf input`)
- [ ] Clé SSH ajoutée sur GitHub
- [ ] Projet cloné dans `/home/<utilisateur>/...`
- [ ] Éditeur connecté à WSL
- [ ] Claude Code installé et connecté
- [ ] Ports `8000`, `8080`, `8025`, `3306` libres

---

## 12 — Dépannage

| Problème | Solution |
|---|---|
| `wsl --install` : erreur de virtualisation (`0x80370102`) | Activer VT-x / AMD-V dans le BIOS, puis relancer. |
| `docker: command not found` dans Ubuntu | Lancer Docker Desktop et vérifier **Resources → WSL integration** (Ubuntu cochée). Fermer/rouvrir le terminal. |
| `permission denied ... docker.sock` | Redémarrer Docker Desktop ; en dernier recours `wsl --shutdown` dans PowerShell. |
| Docker / Symfony très lents | Le projet est sous `/mnt/c/...` : le déplacer dans `~/` (section 8). |
| `port is already allocated` | Un autre logiciel utilise le port (WAMP, XAMPP, Skype…). L'arrêter. Sous PowerShell : `netstat -ano \| findstr :8000`. |
| Fichiers modifiés partout dans `git status` (fins de ligne) | Vérifier `core.autocrlf input` (Ubuntu) ; ne pas éditer le dépôt avec Git pour Windows réglé en `true`. |
| WSL consomme trop de RAM (`vmmem`) | Créer `.wslconfig` (section 2) puis `wsl --shutdown`. |
| `add-apt-repository: command not found` | `sudo apt install -y software-properties-common` puis relancer. |
| `php -v` affiche encore 8.3 | `sudo update-alternatives --config php` et choisir 8.4 (ou 8.5). |
| `symfony check:requirements` signale une extension manquante | Installer le paquet correspondant, ex. : `sudo apt install -y php8.4-intl`. |
| Horloge décalée dans WSL après une mise en veille | `sudo hwclock -s` ou `wsl --shutdown`. |

---

[Retour à l'accueil](README.md) · [Plan pour débuter](PLAN.md)
