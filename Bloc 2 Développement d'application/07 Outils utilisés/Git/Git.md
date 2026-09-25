# 1. Présentation

**Git** est un logiciel de gestion de versions.

Il permet notamment de :

- conserver l'historique des modifications d'un projet ;
- savoir qui a effectué une modification ;
- revenir à une version précédente ;
- travailler à plusieurs sur un même projet ;
- comparer différentes versions d'un fichier ;
- synchroniser un projet avec un serveur distant tel que GitHub ou GitLab.

Git est un système **décentralisé**.

Cela signifie que chaque dépôt local contient l'historique du projet. Le dépôt GitHub n'est donc pas l'endroit où Git stocke l'unique copie de l'historique : chaque dépôt cloné possède lui-même une copie de cet historique.

Il ne faut pas confondre **Git** et **GitHub**.

**Git** est le logiciel de gestion de versions installé sur votre ordinateur.

**GitHub** est un service en ligne permettant notamment d'héberger des dépôts Git et de travailler à plusieurs.

On peut utiliser Git sans GitHub.

Dans un projet utilisant GitHub, on retrouve généralement :

```text
Ordinateur de l'étudiant
        │
        │ Git
        ▼
Dépôt local
        │
        │ git push / git pull
        ▼
Dépôt GitHub
```

# 2. Les trois zones de Git

Un dépôt Git utilise trois zones principales.

```text
┌───────────────────────┐
│ Répertoire de travail │
│   Working Directory   │
└───────────┬───────────┘
            │ git add
            ▼
┌───────────────────────┐
│         Index         │
│    Staging Area       │
└───────────┬───────────┘
            │ git commit
            ▼
┌───────────────────────┐
│     Dépôt local       │
│      Repository       │
│       .git/           │
└───────────────────────┘
```

## 2.1 Répertoire de travail

C'est le dossier dans lequel vous travaillez réellement.

Vous pouvez :

- créer des fichiers ;
- modifier des fichiers ;
- supprimer des fichiers ;
- renommer des fichiers.
    
## 2.2 Zone d'index — Staging Area

L'index est une zone de préparation.

La commande :

```bash
git add fichier.java
```

indique à Git que la version actuelle de `fichier.java` doit être incluse dans le prochain commit.

La commande :

```bash
git add -A
```

prépare toutes les modifications du dépôt.

> **Conseil :** avant un commit, utilisez `git status` afin de vérifier précisément ce qui va être enregistré.

## 2.3 Dépôt local

Le dépôt local est situé dans le répertoire :

```text
.git/
```

Il contient notamment l'historique des commits.

La commande :

```bash
git commit
```

enregistre dans cet historique le contenu préparé dans l'index.

# 3. L'état des fichiers

Un fichier peut être :

### Non suivi — `untracked`

Le fichier existe dans le répertoire de travail mais Git ne le suit pas encore.

```text
?? nouveau-fichier.java
```

Il faut utiliser :

```bash
git add nouveau-fichier.java
```

### Modifié — `modified`

Le fichier était déjà suivi par Git mais son contenu a changé depuis le dernier commit.

```text
modified: fichier.java
```

Il faut utiliser :

```bash
git add fichier.java
```

pour préparer cette modification.

### Indexé — `staged`

La modification a été ajoutée à l'index et sera incluse dans le prochain commit.

```text
Changes to be committed
```

### Validé — `committed`

La modification a été enregistrée dans un commit du dépôt local.

## 3.1 Visualiser l'état du projet

La commande à retenir est :

```bash
git status
```

C'est **la commande à utiliser avant et après les opérations importantes**.

# 4. Première configuration de Git

Après l'installation de Git, il est recommandé de vérifier sa configuration avant de la modifier.

## 4.1 Afficher la configuration actuelle

```bash
git config --list --show-origin --show-scope
```

Cette commande permet notamment de connaître :

- les valeurs configurées ;
- leur origine ;
- leur niveau de configuration.

Les trois niveaux principaux sont :

|Niveau|Commande|Portée|
|---|---|---|
|Système|`--system`|Tout l'ordinateur|
|Global|`--global`|Utilisateur Windows|
|Local|`--local`|Dépôt Git courant|

Une configuration locale est prioritaire sur une configuration globale.


## 4.2 Configuration recommandée pour un poste étudiant

Pour un poste personnel, certains paramètres peuvent être configurés globalement.

### Branche par défaut

```bash
git config --global init.defaultBranch main
```

Cette commande définit `main` comme nom de branche par défaut lors d'un `git init`.

### Gestion des fins de ligne sous Windows

```bash
git config --global core.autocrlf true
```

Cette configuration permet à Git de gérer automatiquement les différences de fins de ligne entre Windows et les dépôts Git.

### Éditeur de texte

Par exemple, avec Notepad++ :

```bash
git config --global core.editor "C:/Program Files/Notepad++/notepad++.exe"
```


## 4.3 Identité utilisée pour les commits

Chaque commit contient le nom et l'adresse e-mail de son auteur.

Sur un poste personnel, on peut utiliser :

```bash
git config --global user.name "Prénom Nom"
git config --global user.email "prenom.nom@example.fr"
```

### Cas des postes partagés ou de plusieurs comptes

Dans un contexte pédagogique, il peut être préférable de définir l'identité **localement dans chaque projet** :

```bash
git config --local user.name "Prénom Nom"
git config --local user.email "prenom.nom@example.fr"
```

Cette configuration ne concerne alors que le dépôt courant.

> **Important :** l'adresse e-mail utilisée dans les commits doit idéalement être une adresse associée au compte GitHub de l'étudiant, ou une adresse `noreply` fournie par GitHub.

## 4.4 Vérifier l'identité utilisée

Dans un dépôt :

```bash
git config user.name
```

```bash
git config user.email
```

Pour connaître l'origine de la configuration :

```bash
git config --list --show-origin --show-scope
```

# 5. Authentification auprès de GitHub

Lorsque Git communique avec GitHub, il doit être autorisé à accéder au dépôt.

Avec une installation récente de Git pour Windows, **Git Credential Manager** peut gérer l'authentification et mémoriser les informations nécessaires.

Lors de la première connexion à GitHub, une authentification peut être demandée dans le navigateur.

> **Important :** le mot de passe habituel du compte GitHub n'est pas utilisé comme mot de passe Git pour les opérations HTTPS. L'authentification moderne utilise notamment des mécanismes tels que OAuth ou des jetons.

Les informations d'identification peuvent être gérées depuis le **Gestionnaire d'identifiants Windows**.

Chemin :

**Panneau de configuration → Comptes d'utilisateurs → Gestionnaire d'identifiants**

Si nécessaire, une entrée GitHub peut être supprimée afin de forcer une nouvelle authentification.

# 6. Récupérer un projet existant

C'est généralement la première opération réalisée dans un TP.

Pour récupérer un projet GitHub :

```bash
git clone https://github.com/utilisateur/projet.git
```

Exemple :

```bash
git clone https://github.com/professeur/evaluation.git
```

Git crée alors un répertoire contenant :

- les fichiers du projet ;
- le dépôt Git local ;
- l'historique des commits ;
- une liaison avec le dépôt distant.

Entrer dans le projet :

```bash
cd evaluation
```

Vérifier :

```bash
git status
```

Puis :

```bash
git remote -v
```


# 7. Initialiser un nouveau dépôt local

Cette procédure est utilisée lorsqu'un projet existe déjà sur l'ordinateur mais n'est pas encore un dépôt Git.

Se placer dans le projet :

```bash
cd chemin/vers/mon-projet
```

Initialiser Git :

```bash
git init
```

Git crée alors le répertoire :

```text
.git/
```

Vérifier :

```bash
git status
```


## 7.1 Ajouter un fichier `.gitignore`

Le fichier `.gitignore` permet d'indiquer à Git quels fichiers ou répertoires ne doivent pas être suivis.

Exemple pour un projet Java/IntelliJ :

```text
.idea/
*.class
target/
```

Il est fortement recommandé de créer le `.gitignore` **avant le premier `git add`**.

## 7.2 Premier commit

Ajouter les fichiers :

```bash
git add -A
```

Créer le premier commit :

```bash
git commit -m "Initialisation du projet"
```

# 8. Le cycle normal de travail

Le cycle quotidien de travail avec Git est généralement :

```text
Modifier
   │
   ▼
git status
   │
   ▼
git add
   │
   ▼
git status
   │
   ▼
git commit
   │
   ▼
git push
```

---

## 8.1 Modifier les fichiers

Travaillez normalement dans votre IDE ou votre éditeur.

Après avoir effectué vos modifications :

```bash
git status
```

Cette commande permet de voir ce qui a changé.

## 8.2 Ajouter les modifications à l'index

Pour un fichier précis :

```bash
git add fichier.java
```

Pour plusieurs fichiers :

```bash
git add fichier1.java fichier2.java
```

Pour toutes les modifications du dépôt :

```bash
git add -A
```

## 8.3 Vérifier ce qui sera commit

Après le `git add` :

```bash
git status
```

Les fichiers préparés apparaissent dans :

```text
Changes to be committed
```

> **Bonne pratique :** ne faites pas systématiquement `git add -A` sans regarder le résultat. Vérifiez toujours `git status`, notamment avant un commit important.


## 8.4 Créer le commit

```bash
git commit -m "Description de la modification"
```

Exemple :

```bash
git commit -m "Ajout de la gestion des utilisateurs"
```

Un commit doit décrire clairement ce qu'il apporte.

Éviter :

```text
modif
test
truc
correction
```

Préférer :

```text
Ajout de la validation du formulaire
Correction du calcul du total
Ajout de la gestion des utilisateurs
```

## 8.5 Envoyer les commits sur GitHub

Après un commit local :

```bash
git push
```

Les commits sont alors envoyés vers le dépôt distant.

Si la branche n'est pas encore associée au dépôt distant :

```bash
git push -u origin main
```

L'option `-u` crée une relation de suivi entre la branche locale et la branche distante.

Par la suite, un simple :

```bash
git push
```

suffit généralement.

# 9. Envoyer les modifications sur GitHub

## 9.1 Vérifier le dépôt distant

```bash
git remote -v
```

Exemple :

```text
origin  https://github.com/verghote/evaluation.git (fetch)
origin  https://github.com/verghote/evaluation.git (push)
```

`origin` est simplement le nom donné au dépôt distant.

---

## 9.2 Ajouter manuellement un dépôt distant

Si aucun dépôt distant n'est configuré :

```bash
git remote add origin https://github.com/verghote/evaluation.git
```

Puis :

```bash
git push -u origin main
```


## 9.3 Créer son dépôt GitHub à partir d'un projet local

Avec GitHub CLI, il est possible de simplifier cette opération :

```bash
gh repo create evaluation --public --source=. --push
```

Cette commande :

1. crée le dépôt GitHub ;
2. utilise le dépôt local courant ;
3. configure le dépôt distant ;
4. pousse les commits existants.
### Exemple

Depuis le projet local :

```bash
gh repo create evaluation --public --source=. --push
```

Si l'utilisateur GitHub connecté est `verghote`, le dépôt créé sera :

```text
verghote/evaluation
```

> **Attention :** si le projet possède déjà un `origin` pointant vers le dépôt du professeur, vérifiez-le avec `git remote -v` avant cette opération.

# 10. Récupérer les modifications depuis GitHub

## 10.1 `git pull`

La commande :

```bash
git pull
```

permet de récupérer les modifications du dépôt distant et de les intégrer dans la branche locale.

Le cycle peut donc être :

```text
        GitHub
          │
       git pull
          ▼
     Dépôt local
          │
     modifications
          │
     git add/commit
          │
       git push
          ▼
        GitHub
```


## 10.2 Avant de commencer à travailler

Dans un projet partagé, il est recommandé de récupérer les dernières modifications :

```bash
git pull
```

Puis :

```bash
git status
```


## 10.3 `git fetch`

La commande :

```bash
git fetch origin
```

récupère les informations du dépôt distant **sans modifier directement la branche de travail actuelle**.

Elle est utile pour examiner les modifications distantes avant de décider comment les intégrer.

Différence simplifiée :

```text
git fetch
→ récupère les informations distantes

git pull
→ récupère puis intègre les modifications
```

# 11. Consulter l'historique

Afficher l'historique :

```bash
git log
```

Version compacte :

```bash
git log --oneline
```

Exemple :

```text
a31f8c2 Ajout de la gestion des utilisateurs
d8a21bc Correction du formulaire
7f42a10 Initialisation du projet
```

Chaque commit possède un identifiant appelé **hash**.

Il est généralement possible d'utiliser une forme abrégée du hash, par exemple :

```text
a31f8c2
```

plutôt que l'identifiant complet.

## 11.1 Afficher les modifications d'un commit

```bash
git show a31f8c2
```

# 12. Annuler une modification

Il faut distinguer plusieurs situations.

## 12.1 Annuler une modification non indexée

Si un fichier a été modifié mais n'a pas encore été ajouté avec `git add`, on peut restaurer sa dernière version validée :

```bash
git restore nomFichier
```

> ⚠️ Les modifications locales de ce fichier seront perdues.

## 12.2 Retirer un fichier de l'index

Si un fichier a été ajouté par erreur :

```bash
git restore --staged nomFichier
```

Le fichier reste modifié dans le répertoire de travail, mais il ne fera plus partie du prochain commit.

## 12.3 Annuler un commit déjà publié

Si un commit a déjà été envoyé sur GitHub, il est généralement préférable d'utiliser :

```bash
git revert <hash>
```

Exemple :

```bash
git revert a31f8c2
```

Git crée alors **un nouveau commit qui annule les modifications du commit indiqué**.

Cette méthode est préférable lorsque l'historique est déjà partagé.

> Contrairement à certaines commandes destructives, `git revert` ne supprime pas l'ancien commit de l'historique.

# 13. Modifier le dernier commit

Si vous venez de créer un commit mais avez oublié une modification, vous pouvez compléter ce commit.

Modifier le fichier puis :

```bash
git add fichier.java
```

Puis :

```bash
git commit --amend
```

ou :

```bash
git commit --amend -m "Nouveau message du commit"
```

Cette opération modifie le dernier commit au lieu d'en créer un nouveau.

> ⚠️ Si le commit a déjà été envoyé sur une branche partagée, soyez prudent : `--amend` réécrit l'historique.

# 14. Retirer un fichier du suivi Git

Supposons qu'un fichier ait été ajouté par erreur au dépôt.

Pour supprimer le fichier du dépôt **et du disque** :

```bash
git rm nomFichier
```

Pour supprimer le fichier du suivi Git mais **le conserver sur le disque** :

```bash
git rm --cached nomFichier
```

Exemple :

```bash
git rm -r --cached .idea
```

Cette commande est utile lorsqu'un répertoire comme `.idea/` a été ajouté par erreur.

Après avoir ajouté `.idea/` dans `.gitignore` :

```bash
git rm -r --cached .idea
git commit -m "Suppression de .idea du dépôt"
git push
```

# 15. Les fichiers et répertoires ignorés

Le fichier :

```text
.gitignore
```

permet de demander à Git de ne pas suivre certains fichiers.

Exemple :

```text
.idea/
target/
*.class
.env
```

Cela permet notamment d'éviter de versionner :

- les fichiers générés par un IDE ;
- les fichiers compilés ;
- les fichiers temporaires ;
- les fichiers contenant des informations sensibles.

> **Attention :** `.gitignore` n'efface pas un fichier déjà suivi par Git. Si le fichier est déjà dans l'historique, il faut d'abord le retirer du suivi avec `git rm --cached`.

# 16. Les répertoires vides

Git ne versionne pas directement les répertoires.

Un répertoire vide n'apparaît donc pas dans le dépôt.

Si un répertoire doit absolument être conservé, on peut y placer un fichier tel que :

```text
.gitkeep
```

Par exemple :

```text
new_directory/
└── .gitkeep
```

Puis :

```bash
git add new_directory/.gitkeep
git commit -m "Ajout du répertoire new_directory"
git push
```

`.gitkeep` n'est pas une fonctionnalité particulière de Git : c'est simplement une convention utilisée pour placer un fichier dans un répertoire autrement vide.

# 17. Modifier le dépôt distant

Il peut être nécessaire de modifier l'adresse du dépôt distant.

Afficher l'adresse actuelle :

```bash
git remote -v
```

Modifier l'adresse :

```bash
git remote set-url origin https://github.com/utilisateur/nouveau-projet.git
```

Vérifier :

```bash
git remote -v
```

# 18. Dépôts locaux et dépôts distants

Il est important de comprendre qu'un projet peut posséder plusieurs dépôts distants.

Afficher les dépôts distants :

```bash
git remote -v
```

Exemple :

```text
origin  https://github.com/verghote/evaluation.git (fetch)
origin  https://github.com/verghote/evaluation.git (push)
```

Le nom `origin` est une convention. Il est possible d'avoir d'autres noms.

Par exemple :

```text
origin
upstream
```

Dans certains projets pédagogiques, on peut avoir :

```text
upstream → dépôt du professeur
origin   → dépôt personnel de l'étudiant
```

Cette organisation permet de récupérer les modifications du dépôt du professeur tout en envoyant son propre travail vers son dépôt personnel.

# 19. Que faire en cas d'erreur lors d'un `push` ?

## 19.1 Erreur `non-fast-forward`

Un message tel que :

```text
Updates were rejected because the tip of your current branch is behind
```

signifie généralement que le dépôt distant contient des commits que votre dépôt local ne possède pas.

### Première réaction

**Ne pas utiliser immédiatement `git push --force`.**

Commencer par :

```bash
git pull
```

puis résoudre les éventuels conflits.

## 19.2 Conflit de fusion

Git peut signaler :

```text
CONFLICT
```

Dans ce cas :

1. ouvrir les fichiers concernés ;
    
2. rechercher les marqueurs de conflit :
    

```text
<<<<<<< HEAD
...
=======
...
>>>>>>> ...
```

3. choisir ou combiner les modifications ;
    
4. supprimer les marqueurs ;
    
5. ajouter les fichiers corrigés :
    

```bash
git add fichier.java
```

6. terminer la fusion :
    

```bash
git commit
```

Puis :

```bash
git push
```

## 19.3 Historique sans lien

Une erreur telle que :

```text
fatal: refusing to merge unrelated histories
```

signifie que Git considère que les deux historiques n'ont pas de relation commune.

Cela peut arriver notamment lorsque :

- le dépôt local a été initialisé indépendamment ;
- le dépôt GitHub a été initialisé séparément avec un README ;
- puis les deux ont été reliés après coup.

Dans ce cas, **ne forcez pas immédiatement le push**.

Commencez par examiner la situation :

```bash
git status
git log --oneline --graph --all
git remote -v
```

Selon le contexte, on pourra soit fusionner les historiques, soit repartir proprement d'un dépôt distant vide.

## 19.4 `git push --force`

La commande :

```bash
git push --force
```

réécrit l'historique du dépôt distant.

Elle peut donc **écraser des commits présents sur GitHub**.

> ⚠️ À éviter dans le travail quotidien et particulièrement sur une branche partagée.

Si une réécriture d'historique est réellement nécessaire, une forme plus sûre est :

```bash
git push --force-with-lease
```

Elle vérifie que le dépôt distant n'a pas changé depuis votre dernière récupération.

> **Règle pédagogique :** si vous ne savez pas pourquoi `--force` est nécessaire, ne l'utilisez pas.

# 20. Fusionner plusieurs commits

Il peut être utile de regrouper plusieurs petits commits en un seul. Cette opération est appelée **squash**.

Par exemple :

```text
Ajout fichier
Correction
Petite correction
Correction finale
```

peut être regroupé en :

```text
Ajout de la fonctionnalité X
```

## 20.1 Rebase interactif

Pour les cinq derniers commits :

```bash
git rebase -i HEAD~5
```

Git ouvre alors un éditeur contenant quelque chose comme :

```text
pick abc1234 Premier commit
pick def5678 Deuxième commit
pick ghi9101 Troisième commit
pick jkl1121 Quatrième commit
pick mno3141 Cinquième commit
```

Pour fusionner les commits suivants avec le premier :

```text
pick abc1234 Premier commit
squash def5678 Deuxième commit
squash ghi9101 Troisième commit
squash jkl1121 Quatrième commit
squash mno3141 Cinquième commit
```

On peut également utiliser :

```text
s
```

à la place de :

```text
squash
```

Git demandera ensuite de choisir le message du commit final.

## 20.2 Attention à la réécriture de l'historique

Un `rebase` modifie les identifiants des commits.

Si les commits ont déjà été publiés sur GitHub, il faudra généralement mettre à jour le dépôt distant avec :

```bash
git push --force-with-lease
```

> ⚠️ Cette opération doit être évitée sur une branche utilisée par plusieurs personnes.

Pour un étudiant travaillant seul sur son dépôt personnel, le risque est plus faible, mais il faut tout de même comprendre que l'historique distant est réécrit.

# 21. Commandes essentielles à retenir

Pour un étudiant, les commandes suivantes couvrent la grande majorité du travail quotidien.

## Configuration

```bash
git config --list --show-origin --show-scope

git config --local user.name "Prénom Nom"
git config --local user.email "prenom.nom@example.fr"
```

## Récupérer un projet

```bash
git clone URL
```

## Vérifier l'état du projet

```bash
git status
```

## Préparer les modifications

```bash
git add fichier
```

ou :

```bash
git add -A
```

## Créer un commit

```bash
git commit -m "Description"
```

## Envoyer sur GitHub

```bash
git push
```

## Récupérer les modifications

```bash
git pull
```

## Consulter l'historique

```bash
git log --oneline
```

## Voir les dépôts distants

```bash
git remote -v
```

## Récupérer les informations distantes sans fusionner

```bash
git fetch
```

## Annuler un commit déjà publié

```bash
git revert <hash>
```

## Retirer un fichier de l'index

```bash
git restore --staged fichier
```

## Annuler une modification locale

```bash
git restore fichier
```

## Créer un dépôt GitHub depuis le projet local avec `gh`

```bash
gh repo create nom-du-projet --public --source=. --push
```

# 22. Le cycle à retenir

Dans la majorité des TP, le cycle de travail sera simplement :

```text
┌──────────────────────────┐
│  1. Récupérer le projet  │
│       git clone          │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  2. Récupérer les        │
│     dernières versions   │
│       git pull           │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  3. Modifier le projet   │
│     IDE / éditeur        │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  4. Vérifier             │
│       git status         │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  5. Préparer             │
│       git add            │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  6. Valider              │
│       git commit         │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  7. Envoyer sur GitHub   │
│       git push           │
└──────────────────────────┘
```

### La règle des 4 commandes

Pour le travail quotidien, retenez surtout :

```bash
git status
git add -A
git commit -m "Description de la modification"
git push
```

Et avant de commencer à travailler sur un projet partagé :

```bash
git pull
```

# 23. Bonnes pratiques

### Toujours vérifier avant un commit

```bash
git status
```

### Faire des commits réguliers

Un commit doit correspondre à une modification cohérente.

### Écrire des messages de commit explicites

Préférer :

```text
Correction du calcul du prix TTC
```

à :

```text
modif
```

### Ne jamais versionner de secrets

Ne jamais ajouter au dépôt :

- mots de passe ;
- clés API ;
- tokens ;
- fichiers `.env` contenant des secrets ;
- certificats ou clés privées.

Utiliser `.gitignore` lorsque cela est approprié.

### Avant un `push`

Vérifier :

```bash
git status
git log --oneline -5
git remote -v
```

### Ne pas utiliser `--force` sans comprendre

La commande :

```bash
git push --force
```

peut supprimer de l'historique distant des commits qui ne sont plus accessibles par la branche.

Préférer, lorsque la réécriture est réellement nécessaire :

```bash
git push --force-with-lease
```

# 24. Résumé

Git repose sur une idée simple :

```text
Fichiers
   │
   │ git add
   ▼
Index
   │
   │ git commit
   ▼
Dépôt local
   │
   │ git push
   ▼
Dépôt distant
```

Et pour récupérer le travail des autres :

```text
Dépôt distant
      │
      │ git pull
      ▼
Dépôt local
```

Le réflexe principal d'un étudiant doit être :

```bash
git status
```

avant toute opération importante.

Puis, dans le cycle normal :

```bash
git add
git commit
git push
```

et, avant de commencer à travailler sur un projet partagé :

```bash
git pull
```

> **À retenir :** Git permet de gérer l'historique localement. GitHub permet notamment de partager et synchroniser cet historique avec d'autres personnes.