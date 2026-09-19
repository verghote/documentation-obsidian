
Une branche permet de partir sur une autre version afin de tester la mise en place d'une nouvelle fonctionnalité. La branche initiale `master` (maintenant `main`) peut continuer à évoluer de façon indépendante.

## Principales commandes

### Création d'une branche

```bash
git branch nomBranche
```

### Création d'une branche en se positionnant dessus

```bash
git switch -c nouvelleBranche
```

### Sélection d'une branche

Deux commandes possibles :

```bash
git switch nomBranche
```

ou

```bash
git checkout nomBranche
```

### Création et sélection d'une branche

```bash
git checkout -b nomBranche
```

### Connaître les branches

L'astérisque (`*`) indique la branche active.

```bash
git branch
```

> **Remarque :** Depuis juin 2020, de nombreuses entreprises technologiques ont souhaité abandonner certains termes jugés non inclusifs tels que « maître », « esclave », « liste noire » et « liste blanche ».
>
> Depuis septembre 2020, GitHub et Git utilisent donc `main` comme nom de branche par défaut à la place de `master`.

---

# Fusion de branches

À un moment donné, il faut réintégrer les branches dans la branche principale (`main`) afin d'intégrer les développements dans la version déployée.

## Fusion de la branche `nomBranche` dans la branche `main`

Se placer sur la branche `main` :

```bash
git checkout main
git merge nomBranche
```

## Mettre à jour une branche avec les modifications de `main`

Cette méthode est utile lorsque l'on souhaite continuer à travailler sur `nomBranche` tout en récupérant les nouvelles fonctionnalités développées dans `main`.

```bash
git checkout main
git pull origin main

git checkout nomBranche
git merge main

git add .
git commit -m "Fusion de main dans nomBranche"
git push origin nomBranche
```

### Explication des commandes

| Commande | Action |
|-----------|---------|
| `git checkout main` | Basculer sur la branche `main` |
| `git pull origin main` | Récupérer les dernières modifications du dépôt distant |
| `git checkout nomBranche` | Basculer vers la branche de travail |
| `git merge main` | Fusionner les modifications de `main` |
| `git add` / `git commit` | Valider localement les changements |
| `git push origin nomBranche` | Envoyer les changements vers le dépôt distant |

---

# Suppression d'une branche

## Supprimer une branche locale intégrée

```bash
git branch -d nomBranche
```

## Supprimer une branche locale non intégrée

```bash
git branch -D nomBranche
```

## Supprimer une branche distante

```bash
git push origin --delete nomBranche
```

---

# Résoudre un conflit

Il est possible que deux branches modifient le même fichier.

Dans ce cas, un conflit se produit lors de la fusion. La commande `git merge` génère alors une erreur et demande de résoudre le conflit.

À l'aide de votre éditeur :

1. Identifiez les parties en conflit.
2. Choisissez les modifications à conserver.
3. Supprimez les marqueurs de conflit.
4. Enregistrez le fichier.

Une fois le conflit résolu :

```bash
git add .
git commit -m "Résolution du conflit"
```

> **Important :** N'oubliez pas de réaliser un commit afin de valider la résolution du conflit.

---

# Mise à jour d'une branche sans perdre les modifications non enregistrées

Lorsque vous travaillez sur une branche, vous ne pouvez généralement pas changer de branche sans enregistrer vos modifications.

Cependant, il peut arriver que :

- vous soyez en train de développer une fonctionnalité ;
- une correction urgente doive être effectuée ;
- vous ne souhaitiez pas encore valider votre travail en cours.

Git propose une solution : **le stash**.

## Sauvegarder temporairement les modifications

```bash
git stash
```

Cette commande retire les modifications du répertoire de travail tout en les conservant dans une zone temporaire.

Vous pouvez alors effectuer vos corrections urgentes, les valider puis revenir à votre travail initial.

## Restaurer les modifications sauvegardées

```bash
git stash apply
```

Les modifications précédemment sauvegardées sont réinjectées dans votre espace de travail.

### Complément

http://sametmax.com/soyez-relax-faites-vous-un-petit-git-stash/

---

# Mise à jour d’un dépôt distant depuis une branche

La commande reste :

```bash
git push
```

Cependant, lors du premier envoi de la branche vers le dépôt distant :

```bash
git push --set-upstream origin nomBranche
```

> **Remarque :** En utilisant PhpStorm, cette configuration est généralement réalisée automatiquement.

---

# Exemple complet

Création d'une branche `information` dans laquelle on ajoute un répertoire `information` contenant un fichier `index.php`.

```bash
git branch information
git checkout information

# Création du répertoire information
# Création du fichier index.php

git add -A
git commit -m "Ajout du répertoire information"
git push --set-upstream origin information
```

## Observation dans GitHub

On peut voir :

- la branche `main`
- la branche `information`

La seule différence est la présence du répertoire `information` et du commit associé dans la branche `information`.

## Observation dans PhpStorm

Si l'on revient sur la branche `main` :

```bash
git checkout main
```

Le répertoire `information` disparaît physiquement du disque.

Si l'on revient sur la branche `information` :

```bash
git checkout information
```

Le répertoire réapparaît automatiquement.

Cela montre que Git adapte le contenu du répertoire de travail à la branche actuellement sélectionnée.