# Notes d'apprentissage – Vélib' DataOps

## Session 0 : installation de l'environnement

### WSL
- WSL est un vrai Linux (noyau Linux) intégré à Windows, dans une VM légère.
- Utilité : travailler dans le même environnement que les serveurs de production,
  où tournent Docker et les outils DevOps.
- Savoir où je suis :
  - `PS C:\...>` = Windows (PowerShell)
  - `myriamboumedine@...:~$` = Linux (Ubuntu)
- `wsl.exe --shutdown` : redémarre WSL. Le `.exe` lance une commande Windows depuis Linux.

### Docker dans WSL
- Activer l'intégration : Docker Desktop > Settings > Resources > WSL integration > Ubuntu.
- Docker ne marche dans Ubuntu que si Docker Desktop est lancé ("Engine running").

### Erreur : `permission denied ... docker.sock`
- Cause : le fichier `/var/run/docker.sock` n'est accessible qu'à root et au groupe `docker`.
- Diagnostic :
  - `ls -l /var/run/docker.sock` : propriétaire, groupe et droits du fichier
  - `groups` : mes groupes
- Correction : `sudo usermod -aG docker $USER`, puis redémarrer WSL.
- Les groupes ne sont pris en compte qu'à la reconnexion.

### Erreur : `daemon is running` / `no such file or directory`
- Cause : Docker Desktop était arrêté (le redémarrage de WSL l'avait éteint).
- Correction : lancer Docker Desktop et attendre "Engine running".

### Utilisateurs et droits Linux
- `root` = administrateur. `sudo` = exécuter une commande en tant qu'administrateur.
- Mot de passe Ubuntu oublié : `wsl -u root` (PowerShell), puis `passwd <utilisateur>`.
- Droits : `rw-r--r--` = propriétaire / groupe / autres (r = lire, w = écrire, x = exécuter).
- `chmod 600 fichier` = lisible et modifiable uniquement par moi (`rw-------`).

### Commandes de base
- `pwd` : où suis-je
- `cd dossier` : se déplacer
- `mkdir -p dossier` : créer un dossier
- `ls -l` : lister avec les détails
- `cat fichier` : afficher un fichier
- `~` = mon dossier personnel, `.` = dossier courant, `..` = dossier parent
- `code .` : ouvrir VS Code sur le dossier courant (extension WSL obligatoire)
- `Ctrl+C` : interrompre une commande
- `Tab` : autocomplétion (évite les fautes de frappe)

### Clé SSH
- Paire de clés : la clé privée reste sur mon PC (jamais partagée),
  la clé publique (`.pub`) est donnée au service (GitHub).
- Créer : `ssh-keygen -t ed25519 -C "mon@email.com"`
- Tester : `ssh -T git@github.com`
- Première connexion : vérifier l'empreinte du serveur avec celle publiée officiellement.
- `known_hosts` : serveurs connus. Alerte si l'empreinte change (protection man-in-the-middle).
- `~/.ssh/config` : règles par serveur (quelle clé, bastion avec `ProxyJump`...).

### Git
- `git config --global user.name / user.email` : mon identité dans les commits.
- `init.defaultBranch main` : nom de la première branche lors d'un `git init`.

## Leçons retenues
- Lire le message d'erreur : chaque erreur pointe vers une cause précise.
- Vérifier dans quel terminal et quel dossier je suis.
- Diagnostiquer avant de corriger.
- Quand une commande pose une question, répondre à la question, ne pas taper une autre commande.