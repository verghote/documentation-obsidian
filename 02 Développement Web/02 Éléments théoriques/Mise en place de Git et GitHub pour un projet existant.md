
Ce guide explique comment :

1. Initialiser un dépôt Git local à partir d'un projet existant.
    
2. Créer un dépôt GitHub.
    
3. Relier le dépôt local au dépôt GitHub.
    
4. Envoyer le projet sur GitHub.
    
5. Réaliser les mêmes opérations depuis PHPStorm.
    

# Prérequis

- Git installé sur le poste.
- Un compte GitHub.
- PHPStorm installé.
- Un projet déjà présent sur le disque.

# Étape 1 - Vérifier la configuration Git

Ouvrir un terminal et vérifier l'identité Git :

```
git config --global user.name
git config --global user.email
```

Si nécessaire :

```bash
git config --global user.name "Prénom Nom"
git config --global user.email "prenom.nom@exemple.com"
```

# Étape 2 - Initialiser le dépôt Git local

Se placer dans le dossier du projet :

```
cd mon-projet
```

Créer le dépôt Git :

```
git init
```

Vérifier l'état du dépôt :

```
git status
```

Ajouter tous les fichiers :

```
git add .
```

Créer le premier commit :

```
git commit -m "Initialisation du projet"
```


# Étape 3 - Créer le dépôt GitHub

Deux méthodes sont possibles.

## Méthode recommandée : avec GitHub CLI (`gh`)

Depuis le dossier du projet, exécuter :

```
gh repo create mon-projet --public --source=. --remote=origin
```

ou, pour un dépôt privé :

```
gh repo create mon-projet --private --source=. --remote=origin
```

Cette commande :

- crée le dépôt sur GitHub ;
- ajoute automatiquement le dépôt distant `origin` ;
- n'effectue pas le premier `push`.

Vérifier que le dépôt distant a bien été ajouté :

```
git remote -v
```

Résultat attendu :

```
origin  https://github.com/utilisateur/mon-projet.git (fetch)origin  https://github.com/utilisateur/mon-projet.git (push)
```

Le projet peut ensuite être publié avec :

```
git push -u origin main
```

> **Remarque**
> 
> Lors de la première utilisation de `gh`, une authentification est demandée. Elle s'effectue une seule fois avec :
> 
> ```
> gh auth login
> ```
> 
> Le navigateur s'ouvre afin d'autoriser GitHub CLI à accéder au compte GitHub.

## Méthode alternative : depuis le site GitHub

Se connecter à GitHub.

Créer un nouveau dépôt :

- Cliquer sur **New repository**.
- Choisir un nom de dépôt.
- Laisser le dépôt vide :
    - ne pas créer de `README` ;
    - ne pas créer de `.gitignore` ;
    - ne pas créer de licence.

Une fois le dépôt créé, GitHub affiche son URL :

```
https://github.com/utilisateur/mon-projet.git
```
# Étape 4 - Relier le projet local au dépôt GitHub

Si le dépôt a été créé avec **GitHub CLI (`gh`)**, cette étape est déjà réalisée automatiquement.

Si le dépôt a été créé depuis le site GitHub, il faut lier le dépôt distant au dépôt local:

```
git remote add origin https://github.com/utilisateur/mon-projet.git
```

Vérifier la liaison :

```
git remote -v
```

Résultat attendu :

```
origin  https://github.com/utilisateur/mon-projet.git (fetch)
origin  https://github.com/utilisateur/mon-projet.git (push)
```

# Étape 5 - Envoyer le projet sur GitHub

Renommer la branche principale si nécessaire :

```
git branch -M main
```

Envoyer le projet :

```
git push -u origin main
```

Lors du premier push :

- Git demande une authentification GitHub ;
- le navigateur s'ouvre ;
- GitHub demande l'autorisation ;
- le push est ensuite effectué.

L'option `-u` crée l'association entre :

```
branche locale : main
branche distante : origin/main
```

Après le premier push :

```
git remote -v
```

doit afficher :

```
origin  https://github.com/utilisateur/mon-projet.git
```

et le dépôt GitHub doit contenir tous les fichiers du projet.

Le projet est désormais synchronisé entre le dépôt Git local et le dépôt GitHub.


Les prochains envois pourront se faire avec :

```
git push
```

# Cycle de travail quotidien en console

## Vérifier les modifications

```bash
git status
```

## Ajouter les fichiers modifiés

```
git add .
```

## Créer un commit

```
git commit -m "Description des modifications"
```

## Envoyer les modifications

```bash
git push
```

## Récupérer les modifications distantes

```
git pull
```

# Réalisation depuis PHPStorm

## Initialiser Git

Ouvrir le projet dans PHPStorm.

Menu :

```
Git → Create Git Repository...
```

Sélectionner le dossier du projet.

PHPStorm crée automatiquement le dossier `.git`.

## Ajouter les fichiers au dépôt

Menu :

```
Git → Add
```

ou

```text
Clic droit → Git → Add
```

Les fichiers passent dans la zone de staging.

## Créer le premier commit

Menu :

```
Git → Commit...
```

Saisir un message :

```
Initialisation du projet
```

Puis cliquer sur :

```
Commit
```

ou

```
Commit and Push
```

## Associer le dépôt GitHub

Menu :

```
Git → Manage Remotes...
```

Cliquer sur :

```
+
```

Nom :

```
origin
```

URL :

```
https://github.com/utilisateur/mon-projet.git
```

Valider.

## Envoyer le projet sur GitHub

Menu :

```
Git → Push...
```

Premier envoi :

- PHPStorm demande une authentification GitHub ;
- le navigateur s'ouvre ;
- l'autorisation est accordée ;
- le push est exécuté.

## Récupérer les modifications du dépôt distant

Menu :

```
Git → Pull...
```

## Consulter l'historique

Menu :

```
Git → Show History
```

ou

```
Git → Git Log
```

