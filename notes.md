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

### Fin de session 0 : leçons supplémentaires
- Linux est sensible à la casse : `notes.md` et `NOTES.md` sont deux fichiers différents.
- HTTPS ou SSH : vérifier l'adresse du dépôt distant avec `git remote -v`.
  - La changer : `git remote set-url origin git@github.com:<pseudo>/<depot>.git`
  - Avec une clé SSH, toujours choisir l'URL SSH sur GitHub (bouton Code > SSH).
- Premier push : `git push -u origin main` (`-u` relie la branche locale à celle de GitHub).

### Les 4 zones de Git
- Dossier de travail → `git add` → zone de préparation (staging)
  → `git commit` → dépôt local → `git push` → GitHub
- Image : `add` = remplir le carton, `commit` = le fermer avec une étiquette, `push` = l'envoyer.
- Toujours faire `git status` avant `git commit`.
- Attention à `git add .` : il ajoute tout, y compris d'éventuels secrets → utiliser un `.gitignore`.

### Comment fonctionne Docker
- Docker = un client (la commande `docker`) + un daemon (le moteur, qui crée les conteneurs).
- Le client envoie ses ordres au daemon via le socket `/var/run/docker.sock`.
- Accès au socket réservé à root et au groupe `docker`.
- Être dans le groupe `docker` équivaut presque à être administrateur (question d'entretien classique).

## Session 1 – Partie A : le terminal

### Fichiers et dossiers
- `mkdir -p a/b c/d` : créer plusieurs dossiers (et leurs parents) en une commande.
- `cp source destination` : copier.
- `mv ancien nouveau` : déplacer ou renommer.
- `rm fichier` : supprimer un fichier. `rm -r dossier` : supprimer un dossier et son contenu.
  - Pas de corbeille ! Toujours relire le chemin. Jamais de `rm -rf` sur `~` ou `/`.
- `ls -R` ou `tree` : afficher une arborescence.
- `find ~/dossier -iname "nom*"` : chercher un fichier (`-iname` ignore la casse).

### Écrire dans un fichier
- `echo "texte" > fichier` : écrase le fichier.
- `echo "texte" >> fichier` : ajoute une ligne à la fin.
- `wc -l fichier` : compter les lignes.
- `nano fichier` : éditeur de texte dans le terminal.
  - `Ctrl+O` puis Entrée = enregistrer, `Ctrl+X` = quitter.

### Scripts Bash
- La première ligne `#!/bin/bash` (shebang) indique quel programme lit le script.
- Un fichier n'est exécutable que s'il a le droit `x` : `chmod +x script.sh`.
- `-rw-r--r--` devient `-rwxr-xr-x` après `chmod +x`.
- `$(commande)` est remplacé par le résultat de la commande au moment de l'exécution.

### Lire les droits : `-rwxr-xr-x`
- 1er caractère : type (`-` fichier, `d` dossier, `s` socket).
- Puis 3 blocs de 3 : propriétaire / groupe / autres.
- `r` = lire, `w` = écrire, `x` = exécuter, `-` = droit absent.

### Pièges rencontrés
- Plusieurs commandes sur une même ligne : Linux n'en voit qu'une seule. Une commande par ligne.
- Un espace dans un chemin sépare deux éléments : utiliser `/` entre le dossier et le fichier.
- `sudo` demande le mot de passe Ubuntu (rien ne s'affiche à la saisie).

## Session 1 – Partie B : Docker, images et conteneurs

### Image et conteneur
- Image = modèle figé (programme + dépendances + config). Ex : `postgres:16`.
- Conteneur = instance qui tourne, créée à partir d'une image (image = classe, conteneur = objet).
- Le tag (`:16`) = la version de l'image. Sans tag, Docker prend `latest` (à éviter : version imprévisible).
- Les images viennent d'un registry (Docker Hub par défaut) et sont découpées en couches (layers).

### Commandes
- `docker pull postgres:16` : télécharger une image.
- `docker images` : lister les images.
- `docker run -d --name mon-postgres -e POSTGRES_PASSWORD=secret -p 5432:5432 postgres:16`
  - `-d` : en arrière-plan (detached), le terminal reste libre.
  - `--name` : nom du conteneur (pas de la base de données).
  - `-e` : variable d'environnement, la façon standard de configurer un conteneur.
  - `-p MACHINE:CONTENEUR` : relie un port de ma machine à un port du conteneur.
- `docker ps` : conteneurs qui tournent. `docker ps -a` : tous, y compris arrêtés.
- `docker logs mon-postgres` : voir ce qu'affiche le conteneur.
- `docker exec -it mon-postgres psql -U postgres` : lancer une commande dans un conteneur qui tourne.

### Cycle de vie
- `run` : créer et démarrer.
- `stop` / `start` : éteindre et rallumer (le conteneur existe toujours).
- `exec` : entrer dans un conteneur qui tourne (impossible s'il est arrêté).
- `rm` : supprimer le conteneur (il doit être arrêté).
- `rmi` : supprimer une image.

### L'expérience des données
- `stop` puis `start` : la table est toujours là.
- `rm` puis un nouveau `run` : la table a disparu.
- Pourquoi : les données sont stockées dans le conteneur. Supprimer le conteneur = supprimer les données.
- Les conteneurs sont jetables par conception. Solution pour garder les données : les volumes (session 2).

### SQL dans psql
- Textes entre guillemets simples `'Bastille'`. Guillemets doubles = noms de colonnes ou tables.
- Une requête ne s'exécute qu'au `;`.
- `\dt` lister les tables, `\d table` décrire une table, `\q` quitter.

### Pièges
- Port déjà utilisé (`port is already allocated`) : un autre conteneur occupe le port → `docker ps`.
- Toujours faire le ménage après un test : `docker stop` puis `docker rm`.