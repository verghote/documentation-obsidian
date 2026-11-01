
Cette documentation présente les deux méthodes disponibles pour récupérer un projet fourni par l'enseignant et explique la différence entre un projet destiné uniquement au travail en salle de TP et un projet pouvant être poursuivi à domicile.

Deux scripts sont disponibles :

- `recuperer_td.py` : récupérer un projet pour travailler **uniquement en salle de TP** ;
- `recuperer_projet.py` : récupérer un projet et créer **votre propre dépôt GitHub**, afin de pouvoir travailler en salle et à domicile.

Le choix du script est important, car les deux modes de fonctionnement sont différents.

# 1. Deux situations différentes

|Situation|Script|Emplacement du projet|Dépôt GitHub personnel|
|---|---|---|---|
|Travail uniquement en salle de TP|`recuperer_td.py`|`J:\VirtualHostSlam`|❌ Non|
|Travail en salle et/ou à domicile|`recuperer_projet.py`|`J:\VirtualHostSlam`|✅ Oui|

La différence essentielle est la suivante :

- **`recuperer_td.py`** conserve la liaison avec le dépôt GitHub de l'enseignant. Il est destiné aux projets réalisés uniquement dans le cadre des séances de TP.
- **`recuperer_projet.py`** crée une copie personnelle du projet. La liaison avec le dépôt de l'enseignant est supprimée et un dépôt GitHub est créé sur votre propre compte.

Dans les deux cas, le projet est récupéré dans :

```text
J:\VirtualHostSlam
```

# 2. Prérequis

Avant d'utiliser les scripts, **Git** doit être installé et accessible depuis le terminal.

Pour les projets nécessitant un dépôt GitHub personnel, **GitHub CLI (`gh`)** doit également être installé et configuré.

Les principaux outils utilisés sont :

- **Git** : gestion du projet et de son historique ;
- **GitHub** : hébergement du dépôt distant ;
- **GitHub CLI (`gh`)** : création du dépôt GitHub personnel ;
- **PhpStorm** : édition du projet et terminal intégré.

Pour vérifier que Git est disponible :

```text
git --version
```

Pour vérifier que GitHub CLI est disponible :

```text
gh --version
```

# 3. Situation 1 — Projet réalisé uniquement en salle de TP

## Utiliser `recuperer_td.py`

Le script `recuperer_td.py` est prévu lorsqu'un projet doit être réalisé **uniquement sur les ordinateurs de l'établissement**.

Le projet est récupéré dans :

```text
J:\VirtualHostSlam
```

Le dépôt GitHub de l'enseignant reste le dépôt distant du projet.

Voir la description complète de ce script dans  : [[04 Outils d'administration de l'environnement de développement#3. Récupérer un projet pour un travail uniquement en salle de TP]]
## 3.1. Que faire si `recuperer_td.py` ne peut pas être lancé ?

Le projet peut être récupéré manuellement avec Git.

Depuis un terminal :

```text
git clone https://github.com/verghote/NOM_DU_PROJET.git
```

Par exemple : git clone https://github.com/verghote/consultation.git

Vous pouvez ensuite vérifier la liaison vers le dépôt distant avec la commande  : **git remote -v**

```text
origin https://github.com/verghote/consultation.git (fetch) 
origin https://github.com/verghote/consultation.git (push)
```

Le dépôt doit pointer vers celui de l'enseignant.

Si par erreur la liaison est perdue ou supprimée, il suffit de recréer la liason : **git remote add origin https://github.com/verghote/consultation.git**
## 3.2. Mettre à jour un projet récupéré avec l'utilitaire recuperer_td.py

Cette possibilité est notamment utile lorsqu'une nouvelle version du projet a été publiée par l'enseignant, ou lorsque vous avez été absent.

Le projet local étant  toujours connecté au dépôt de l'enseignant, il est possible de récupérer la dernières versions en acceptant alors de perdre ses propres modifications.

Les commandes suivantes, vont remettre complètement le projet dans l'état de la branche `main` du dépôt de l'enseignant :

```text
git fetch
git reset --hard origin/main
git clean -fd --exclude=.idea
```

## Rôle des commandes

| Commande                      | Rôle                                                                                      |
| ----------------------------- | ----------------------------------------------------------------------------------------- |
| git fetch                     | Récupère les dernières informations du dépôt distant sans modifier les fichiers du projet |
| git reset --hard origin/main  | Replace les fichiers suivis par Git dans l'état de `origin/main`                          |
| git clean -fd --exclude=.idea | Supprime les fichiers et dossiers non suivis par Git, en conservant `.idea`               |

> **Attention :** ces commandes peuvent supprimer vos modifications locales non enregistrées dans Git ainsi que certains fichiers non suivis. Elles doivent être utilisées uniquement lorsque vous souhaitez remettre le projet dans l'état fourni par l'enseignant.

La lancement de ces trois commandes Git peut se faire automatiquement en lançant le script  **reprendre_td.py**
# 4. Situation 2 — Projet réalisable en salle et à domicile

## Utiliser `recuperer_projet.py`

Le script `recuperer_projet.py` est destiné aux projets que vous devez pouvoir poursuivre :

- sur un ordinateur de l'établissement ;
- chez vous ;
- ou alternativement dans les deux environnements.

Dans ce cas, le projet doit disposer de **son propre dépôt GitHub**.

Le script récupère le projet fourni par l'enseignant, puis crée automatiquement votre copie personnelle sur votre GitHub.

Voir la description complète de ce script dans  : [[04 Outils d'administration de l'environnement de développement#4. Récupérer un projet et créer son propre dépôt GitHub]]

# 9. Les opérations réalisées automatiquement

Pour un projet appelé `evaluation`, le fonctionnement correspond essentiellement aux étapes suivantes.

## Récupération du projet

```text
git clone https://github.com/verghote/evaluation.git
```

Le projet possède alors :

```text
origin → verghote/evaluation
```

## Suppression de la liaison avec l'enseignant

```text
git remote remove origin
```

Le dépôt local conserve cependant tout son historique Git.

Il n'est simplement plus associé à un dépôt distant.

## Création du dépôt étudiant

Le script réalise ensuite l'équivalent de :

```text
gh repo create evaluation --public --source=. --push
```

Le dépôt est créé sur le compte GitHub actuellement authentifié.

Le résultat est :

```text
origin → VOTRE_COMPTE/evaluation
```

Le projet peut alors être synchronisé avec votre propre dépôt GitHub.

# 10. Vérifier son dépôt personnel

Après l'exécution de `recuperer_projet.py`, vous pouvez vérifier le dépôt distant avec :

```text
git remote -v
```

Le résultat doit correspondre à votre compte GitHub.

Vous pouvez également vérifier votre compte avec :

```text
gh api user --jq .login
```

Le dépôt distant et le compte affiché doivent correspondre à votre compte étudiant.

# 11. Faire manuellement la même opération

Si `recuperer_projet.py` ne peut pas être exécuté, les opérations peuvent être réalisées manuellement.

## Étape 1 — Récupérer le projet

```text
git clone https://github.com/verghote/evaluation.git
```

## Étape 2 — Entrer dans le projet

```text
cd evaluation
```

## Étape 3 — Vérifier le dépôt distant

```text
git remote -v
```

`origin` doit pointer vers le dépôt de l'enseignant.

## Étape 4 — Supprimer le dépôt distant de l'enseignant

```text
git remote remove origin
```

Vérifiez :

```text
git remote -v
```

Aucun dépôt distant ne doit alors être affiché.

## Étape 5 — Vérifier votre compte GitHub

```text
gh auth status
```

ou :

```text
gh api user --jq .login
```

## Étape 6 — Créer votre dépôt personnel

```text
gh repo create evaluation --public --source=. --push
```

Le dépôt est créé sur votre compte GitHub et le projet local y est envoyé.

## Étape 7 — Vérifier la nouvelle liaison

```text
git remote -v
```

`origin` doit maintenant pointer vers **votre compte GitHub**.

# 12. Travailler avec son dépôt GitHub personnel

Une fois `recuperer_projet.py` exécuté, votre dépôt GitHub devient le point de synchronisation entre les différents ordinateurs utilisés.

```text
                 VOTRE DÉPÔT GITHUB
                         │
              ┌──────────┴──────────┐
              │                     │
          git pull              git pull
              │                     │
              ▼                     ▼
       Ordinateur école        Ordinateur personnel
              │                     │
            travail               travail
              │                     │
          git push              git push
              │                     │
              └──────────┬──────────┘
                         ▼
                 VOTRE GITHUB
```

# 13. À la fin d'une séance

Avant de quitter un ordinateur, enregistrez votre travail dans Git puis envoyez-le sur GitHub.

Dans le terminal de PhpStorm :

```text
git status
git add .
git commit -m "Description du travail réalisé"
git push
```

Par exemple :

```text
git commit -m "Ajout de la gestion des utilisateurs"
```

Le `push` est indispensable si vous souhaitez retrouver votre travail sur un autre ordinateur.

# 14. Reprendre le travail sur un autre ordinateur

Avant de commencer à travailler sur un autre ordinateur, récupérez votre dernière version :

```text
git pull
```

Vous pouvez ensuite travailler normalement.

À la fin de la séance :

```text
git add .
git commit -m "Description du travail réalisé"
git push
```

La règle est donc simple :

```text
Je quitte un ordinateur
        ↓
git add .
        ↓
git commit
        ↓
git push
        ↓
      GITHUB
        ↓
Je change d'ordinateur
        ↓
git pull
        ↓
Je reprends le travail
```

> **Push avant de changer d'ordinateur, pull avant de reprendre le travail.**

# 15. Vérifier l'état du projet

Avant de commencer ou de terminer une séance, la commande suivante est particulièrement utile :

```text
git status
```

Elle permet de connaître :

- les fichiers modifiés ;
- les fichiers ajoutés ;
- les fichiers supprimés ;
- les modifications qui ne sont pas encore dans un commit.

Pour consulter l'historique :

```text
git log
```

Pour vérifier le dépôt distant :

```text
git remote -v
```

# 16. Différence entre les deux méthodes

## `recuperer_td.py`

Le projet reste celui de l'enseignant :

```text
             GITHUB ENSEIGNANT
                    │
                 git clone
                    ↓
             Projet sur J:
                    │
                 origin
                    │
                    ↓
             GITHUB ENSEIGNANT
```

**Objectif :** travailler uniquement en salle de TP.

Le dépôt de l'enseignant reste le dépôt distant du projet.

## `recuperer_projet.py`

Le projet devient votre projet personnel :

```text
             GITHUB ENSEIGNANT
                    │
                 git clone
                    ↓
              Projet local
                    │
          git remote remove origin
                    ↓
          Projet indépendant
                    │
            gh repo create
                    ↓
             GITHUB ÉTUDIANT
                    │
              git push / pull
```

**Objectif :** pouvoir travailler aussi bien en salle qu'à domicile.

# 17. Tableau récapitulatif

|Opération|`recuperer_td.py`|`recuperer_projet.py`|
|---|---|---|
|Vérifier Git|✅|✅|
|Récupérer le projet de l'enseignant|✅|✅|
|Conserver `.git`|✅|✅|
|Conserver l'historique Git|✅|✅|
|Conserver `origin` de l'enseignant|✅|❌|
|Supprimer `origin` de l'enseignant|❌|✅|
|Utiliser GitHub CLI|❌|✅|
|Créer un dépôt GitHub personnel|❌|✅|
|Effectuer le premier `push`|❌|✅|
|Travailler à domicile|❌|✅|
|Utiliser `git pull` sur son dépôt personnel|❌|✅|
|Utiliser `git push` sur son dépôt personnel|❌|✅|

# 18. À retenir

### `recuperer_td.py`

> **Je travaille sur le projet de l'enseignant uniquement en salle de TP.**

Le dépôt local reste connecté au dépôt GitHub de l'enseignant.

```text
origin
   ↓
enseignant
```

### `recuperer_projet.py`

> **Je crée ma propre copie du projet pour pouvoir travailler en salle et à domicile.**

Le dépôt local est déconnecté du dépôt de l'enseignant et connecté à votre propre dépôt GitHub.

```text
origin
   ↓
étudiant
```

Dans les deux cas, la présence du dossier `.git` est normale.

Ce qui change est **le dépôt distant (`origin`) auquel le projet est associé**.

C'est cette différence qui détermine la destination des commandes :

```text
git pull
git push
```

Pour un projet personnel, retenez surtout :

```text
git pull
    ↓
je travaille
    ↓
git add .
    ↓
git commit -m "..."
    ↓
git push
```

**Pull avant de commencer, push avant de changer d'ordinateur.**