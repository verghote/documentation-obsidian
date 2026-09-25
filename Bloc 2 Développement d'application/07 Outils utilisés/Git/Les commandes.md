# Git - Guide pratique

<a id="top"></a>
## Sommaire  
  
- [[#Comprendre les 4 zones de Git|Les 4 zones Git]]  
- [[#Configuration initiale de Git|Configuration]]  
- [[#Création et récupération d'un dépôt|Création et récupération d'un dépôt]]  
- [[#Cycle de travail quotidien|Cycle de travail quotidien]]  
- [[#Consulter l'historique|Historique]]  
- [[#Annuler ou corriger des erreurs|Annuler ou corriger]]  
- [[#La commande git stash|Git stash]]  
- [[#Authentification GitHub|Authentification GitHub]]  
- [[#Résolution des erreurs courantes|Erreurs courantes]]


# Comprendre les 4 zones de Git

Git manipule les fichiers à travers quatre zones distinctes :

```text
Working Directory
       |
       | git add
       v
Staging Area
       |
       | git commit
       v
Repository local
       |
       | git push
       v
Repository distant
```

## Working Directory

C'est votre dossier de travail.

Vous modifiez vos fichiers librement dans cette zone.

## Staging Area

Zone de préparation des modifications.

Les fichiers sont ajoutés avec :

```bash
git add .
```

## Repository local

Les modifications sont enregistrées définitivement dans votre dépôt local :

```bash
git commit -m "Description"
```

## Repository distant

Les commits sont envoyés vers GitHub :

```bash
git push
```

# Configuration initiale de Git

## Visualiser la configuration actuelle

Configuration globale :

```bash
git config --list --global
```

Configuration locale du projet courant :

```bash
git config --list --local
```

Afficher l'origine des paramètres :

```bash
git config --list --show-origin
```

## Définir son identité

Ces informations seront enregistrées dans chaque commit.

Configuration globale :

```bash
git config --global user.name "Guy Verghote"
git config --global user.email "guy.verghote@saint-remi.net"
```

Vérifier :

```bash
git config --global user.name
git config --global user.email
```

Sous Windows, la configuration globale est stockée dans :

```text
C:\Users\VotreNom\.gitconfig
```

Sous Linux/macOS :

```text
~/.gitconfig
```

### Configuration spécifique à un projet

Depuis le dossier du projet :

```bash
git config user.name "Guy Verghote"
git config user.email "guy.verghote@ac-amiens.fr"
```

Cette configuration est enregistrée dans :

```text
.git/config
```

Elle est prioritaire sur la configuration globale.

Si la commande est exécutée en dehors d'un dépôt Git :

```text
fatal: not in a git directory
```

Les informations définies avec `user.name` et `user.email` ne servent pas à l'authentification GitHub.

Elles servent uniquement à identifier l'auteur des commits.

## Définir le nom de la branche par défaut

Historiquement Git utilisait `master`.

La convention actuelle est `main`.

```bash
git config --global init.defaultBranch main
```

## Définir la gestion des retours à la ligne

Sous Windows :

```
git config --global core.autocrlf true
```

Git convertira :

- LF → CRLF lors du checkout ;
- CRLF → LF lors du commit.

Cela permet souvent d'éviter les messages :

```
warning: LF will be replaced by CRLF
```

## Définir Notepad++ comme éditeur par défaut

```
git config --global core.editor "\"C:/Program Files/Notepad++/notepad++.exe\" -multiInst -nosession"
```

Cet éditeur sera utilisé notamment lors :

- d'un commit sans option `-m` ;
- d'un merge ;
- d'un rebase ;
- d'une résolution de conflit ;
- de l'édition de la configuration Git.
## Adopter une stratégie de synchronisation propre

```
git config --global pull.rebase true
```

Cette configuration évite la création de nombreux commits de fusion inutiles lors des `git pull`.
## Définir un dépôt comme sûr

Git vérifie que le propriétaire du dépôt correspond à l'utilisateur courant.

Si nécessaire :

```
git config --global --add safe.directory "D:/MonProjet"
```

Afficher les dépôts déclarés sûrs :

```
git config --global --get-all safe.directory
```
# Création et récupération d'un dépôt

## Créer un dépôt local

```
git init
```
## Associer un dépôt distant

```
git remote add origin https://github.com/utilisateur/repo.git
```

Afficher les dépôts distants :

```
git remote -v
```

Remarque : Git ne vérifie pas immédiatement que le dépôt existe.

La vérification est effectuée lors d'un :

```
git push
git pull
git fetch
```
## Cloner un dépôt existant

```
git clone https://github.com/utilisateur/repo.git
```

# Cycle de travail quotidien

## Vérifier l'état du dépôt

```
git status
```

Obtenir la racine du dépôt Git auquel appartient le répertoire courant.

```
git rev-parse --git-dir
```
## Ajouter les modifications à la zone de préparation

Tous les fichiers :

```
git add .
```

Un seul fichier :

```
git add monfichier.txt
```
## Vérifier les modifications

Modifications non préparées :

```
git diff
```

Modifications déjà préparées :

```bash
git diff --staged
```
## Créer un commit

```
git commit -m "Description du commit"
```
## Corriger le dernier commit

Modifier le message :

```
git commit --amend
```

Ajouter un fichier oublié :

```
git add fichier.txt
git commit --amend --no-edit
```

## Envoyer les modifications

Premier push :

```
git push -u origin main
```

Push suivants :

```
git push
```

## Récupérer les modifications distantes

```
git pull
```

Récupérer sans fusionner :

```
git fetch
```
# Consulter l'historique

Historique détaillé :

```
git log
```

Historique condensé :

```
git log --oneline --graph
```

Obtenir **le tout premier commit du dépôt**, avec sa date et son auteur.

```
git log --reverse --format=fuller
```

# Annuler ou corriger des erreurs

## Annuler des modifications locales

```
git restore fichier.txt
```

Les modifications du fichier sont perdues.
## Retirer un fichier de la zone de préparation

```
git restore --staged fichier.txt
```

Les modifications sont conservées.

## Revenir complètement à la version distante

⚠️ Toutes les modifications locales seront perdues.

```
git fetch origin
git reset --hard origin/main
```
## Avec sauvegarde préalable

Sauvegarder :

```
git stash
```

Réinitialiser :

```
git fetch origin
git reset --hard origin/main
```

Restaurer :

```
git stash pop
```

# La commande git stash

Met temporairement de côté les modifications non commitées.
## Créer un stash

```bash
git stash
```

Avec un commentaire :

```bash
git stash push -m "Travail en cours"
```

## Afficher les stashes

```bash
git stash list
```

## Voir le contenu d'un stash

```bash
git stash show
```

## Réappliquer un stash

```bash
git stash pop
```

Réapplique puis supprime le stash.

Ou :

```
git stash apply
```

Réapplique sans supprimer le stash.

## Supprimer un stash

```
git stash drop
```

## Supprimer tous les stashes

```
git stash clear
```

# Authentification GitHub

Lors du premier push :

```
git push -u origin main
```

Git doit vérifier vos droits d'accès au dépôt.

## Déroulement habituel sous Windows

Si Git Credential Manager est installé :

1. Git ouvre le navigateur.
    
2. Vous vous connectez à GitHub.
    
3. GitHub demande votre autorisation.
    
4. Git reçoit un jeton sécurisé.
    
5. Le push est exécuté automatiquement.
    
## Stockage des informations d'authentification

Sous Windows, les identifiants sont généralement stockés dans le Gestionnaire d'identifiants.

Accès :

```
Windows + R
control

Comptes d'utilisateurs
→ Gestionnaire d'identifiants
→ Identifiants Windows
```


# Résolution des erreurs courantes

## fatal: refusing to merge unrelated histories

Fusionner malgré des historiques différents :

```
git pull origin main --allow-unrelated-histories
```

Après la fusion :

```
git push origin main
```

En cas de conflit :

```
git add fichier_conflit
git commit
```

## fatal: detected dubious ownership in repository

Déclarer le dépôt comme sûr :

```
git config --global --add safe.directory "D:/MonProjet"
```

Vérifier :

```
git config --global --get-all safe.directory
```

