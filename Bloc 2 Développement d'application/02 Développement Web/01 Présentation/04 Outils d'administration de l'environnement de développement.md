Le répertoire **utilitaire** contient plusieurs scripts destinés à simplifier la gestion de l'environnement de développement local.

Ils permettent notamment de :

- récupérer un projet depuis GitHub pour un travail uniquement en salle de TP ;
- récupérer un projet et créer automatiquement votre propre dépôt GitHub ;
- créer, supprimer et gérer les VirtualHosts Apache ;
- sauvegarder une ou plusieurs bases MySQL ;
- restaurer une base MySQL à partir d'une sauvegarde.

# 1. Prérequis

Avant d'utiliser ces outils, l'environnement doit être correctement configuré.

## Environnement Windows

Ces scripts sont prévus pour fonctionner sous **Windows**.

Certains d'entre eux utilisent directement les installations locales de :

- WampServer ;
- Apache ;
- MySQL ;
- Git ;
- GitHub CLI.

Les chemins utilisés par les scripts correspondent à la configuration prévue pour l'environnement de développement.

## Python

Les scripts `.py` nécessitent une installation fonctionnelle de Python.

Pour vérifier que Python est accessible depuis un terminal :

```text
python --version
```

ou :

```text
py --version
```

Si l'une de ces commandes affiche la version de Python installée, Python est accessible depuis le terminal.

# 2. Les différents outils

Les outils sont organisés par fonction.

|Fichier|Emplacement / lancement|Fonction|
|---|---|---|
|`recuperer_td.py`|Terminal|Récupérer un projet public GitHub pour travailler **uniquement en salle de TP**|
|`recuperer_projet.py`|Terminal|Récupérer un projet et créer automatiquement **votre propre dépôt GitHub**|
|`apache.py`|Sous-dossier `apache`|Créer, supprimer et gérer les VirtualHosts Apache|
|`apache.bat`|Sous-dossier `apache`|Lancer `apache.py` avec les droits administrateur|
|`save_database.py`|À copier dans le répertoire personnel|Sauvegarder une ou plusieurs bases MySQL|
|`restore_database.py`|À copier dans le répertoire personnel|Restaurer une base MySQL|

Les deux scripts de récupération répondent à **deux besoins différents** :

- `recuperer_td.py` : travail local en salle de TP ;
- `recuperer_projet.py` : projet personnel pouvant être poursuivi en dehors de la salle de TP.

Le choix du script est donc important.

# 3. Récupérer un projet pour un travail uniquement en salle de TP

## `recuperer_td.py`

Le script `recuperer_td.py` permet de récupérer un projet public depuis le compte GitHub de l'enseignant.

Ce mode est destiné aux projets pour lesquels le travail est réalisé **uniquement sur les ordinateurs de la salle de TP**.

Le programme :

1. vérifie que Git est installé ;
2. récupère automatiquement la liste des dépôts publics ;
3. affiche les projets disponibles ;
4. permet de sélectionner le projet à récupérer ;
5. clone le projet dans :

```text
J:\VirtualHostSlam
```

Le projet reste lié au dépôt GitHub de l'enseignant afin de pouvoir récupérer ultérieurement une nouvelle version du projet.

Par exemple, en cas d'absence ou si vous devez remettre votre projet dans son état d'origine, vous pouvez utiliser les commandes Git indiquées dans la documentation consacrée à **Git et GitHub**.

**Ce mode ne crée pas de dépôt GitHub personnel.**

Il est donc adapté lorsque le projet doit rester un projet fourni par l'enseignant et être utilisé uniquement sur les postes de la salle de TP.
## Fonctionnement

Lorsque vous lancez `recuperer_td.py` :

1. le script vérifie que Git est disponible ;
2. il vérifie que le répertoire `J:\VirtualHostSlam` existe ;
3. il récupère la liste des dépôts publics disponibles ;
4. il affiche les projets proposés ;
5. vous sélectionnez le projet souhaité ;
6. il vérifie que le projet n'existe pas déjà sur `J:` ;
7. il clone le dépôt du projet ;
8. il conserve le dossier `.git` et l'historique Git.

Le projet est alors prêt à être ouvert dans PhpStorm.
## Schéma de fonctionnement

```text
                    GITHUB
              Compte de l'enseignant
                       │
                       │ git clone
                       ▼
              J:\VirtualHostSlam
                       │
                       ▼
                    Projet
                       │
                       │ origin
                       ▼
              Dépôt de l'enseignant
```

Le dépôt local conserve donc un `origin` qui pointe vers le dépôt de l'enseignant.
## Exemple

Si vous sélectionnez le projet `evaluation`, le script réalise essentiellement l'équivalent de :

```text
git clone https://github.com/verghote/evaluation.git J:\VirtualHostSlam\evaluation
```

Le projet possède alors :

```text
origin → dépôt de l'enseignant
```

Vous pouvez vérifier cette information avec :

```text
git remote -v
```

# 4. Récupérer un projet et créer son propre dépôt GitHub

## `recuperer_projet.py`

Le script `recuperer_projet.py` est destiné aux projets que vous devez pouvoir poursuivre **en salle de TP mais également depuis chez vous**.

Dans ce cas, le projet initial fourni par l'enseignant est transformé en un projet personnel.

Le programme automatise les opérations nécessaires pour :

1. récupérer le projet fourni par l'enseignant ;
2. supprimer la liaison avec le dépôt GitHub de l'enseignant ;
3. créer un nouveau dépôt GitHub sur **votre compte personnel** ;
4. établir la liaison entre le projet local et votre nouveau dépôt ;
5. envoyer le projet initial sur votre dépôt personnel.

À l'issue de l'opération, le projet présent sur :

```text
J:\VirtualHostSlam
```

est associé à **votre dépôt GitHub personnel**.

Vous pouvez alors travailler sur le projet :

- en salle de TP ;
- chez vous ;
- sur plusieurs séances ;
- en synchronisant votre travail avec GitHub.

Les commandes `git push` et `git pull` permettent ensuite de synchroniser le travail entre votre ordinateur et votre dépôt GitHub.

## Schéma de fonctionnement

```text
                  GITHUB
            Compte de l'enseignant
                     │
                     │ git clone
                     ▼
               Projet local
                     │
                     │
                     │ suppression de origin
                     ▼
             Projet indépendant
                     │
                     │ gh repo create
                     │ + git push
                     ▼
                GITHUB ÉTUDIANT
                     │
              Votre dépôt personnel
```

L'historique Git du projet est conservé.

En revanche, la liaison avec le dépôt de l'enseignant est supprimée.

Le nouveau `origin` pointe vers votre dépôt GitHub personnel.

Lorsque vous lancez le script :

1. il vérifie que Git est disponible ;
2. il vérifie que GitHub CLI est disponible ;
3. il vérifie l'authentification GitHub ;
4. il identifie le compte GitHub actuellement utilisé ;
5. il récupère la liste des projets publics de l'enseignant ;
6. vous sélectionnez le projet souhaité ;
7. il vérifie que le projet n'existe pas déjà localement ;
8. il clone le projet de l'enseignant ;
9. il conserve l'historique Git ;
10. il supprime le `remote` `origin` de l'enseignant ;
11. il crée un dépôt portant le même nom sur votre compte GitHub ;
12. il associe le projet local à ce nouveau dépôt ;
13. il envoie le projet sur GitHub ;
14. il vérifie que le nouveau `origin` correspond à votre dépôt personnel.

À la fin de l'opération, vous disposez donc d'un projet personnel.

## 4.1 Authentification GitHub

Pour utiliser `recuperer_projet.py`, vous devez être connecté à GitHub avec GitHub CLI.

La connexion peut être réalisée avec :

```text
gh auth login --web --scopes delete_repo
```

Cette commande ouvre le navigateur afin de vous permettre de vous authentifier avec votre compte GitHub.

Une fois connecté, vérifiez l'état de l'authentification avec :

```text
gh auth status
```

Vous pouvez également afficher directement le compte GitHub actuellement utilisé :

```text
gh api user --jq .login
```

Le nom de votre compte GitHub doit être affiché.

> **Important :** vérifiez que vous utilisez bien votre compte GitHub étudiant avant de créer votre dépôt personnel.
# 5. Gestion des VirtualHosts Apache

## Objectif

Le programme `apache.py` permet de gérer les VirtualHosts Apache nécessaires à l'exécution des projets PHP sous WampServer.

Le programme se trouve dans le sous-dossier :

```text
apache
```

Il permet notamment de :

- créer un VirtualHost ;
- supprimer un VirtualHost ;
- lister les VirtualHosts existants ;
- démarrer Apache ;
- arrêter Apache ;
- redémarrer Apache ;
- vérifier l'état d'Apache.

```text
apache.py
│
├── 1. Créer un VirtualHost
│      ├── sauvegarde hosts
│      ├── sauvegarde httpd-vhosts.conf
│      ├── création VHost
│      ├── modification hosts
│      └── redémarrage Apache
│
├── 2. Supprimer un VirtualHost
│      ├── suppression VHost
│      ├── suppression hosts
│      └── redémarrage Apache
│
├── 3. Lister les VirtualHosts
│
└── 4. Gestion Apache
       ├── démarrer
       ├── arrêter
       ├── redémarrer
       └── statut
```

## Structure attendue d'un projet

Avant de créer un VirtualHost, le projet doit impérativement être présent sur le disque `J:`.

Par exemple :

```text
J:\VirtualHostSlam\gestion
```

Le projet doit contenir un sous-répertoire :

```text
J:\VirtualHostSlam\gestion\public
```

Le répertoire `public` constitue la **racine Web** du VirtualHost.

La structure du projet est donc par exemple :

```text
J:\VirtualHostSlam\gestion
│
├── public
│   ├── index.php
│   └── ...
│
├── src
├── vendor
├── composer.json
└── ...
```

Apache utilisera directement :

```text
J:\VirtualHostSlam\gestion\public
```




## 5.1. Création d'un VirtualHost avec `apache.py`

Pour créer un VirtualHost, lancer :

```text
apache\apache.bat
```

en utilisant **Exécuter en tant qu'administrateur**.

Le programme affiche alors le menu de gestion d'Apache.

Choisir :

```text
1. Créer un VirtualHost
```

Le programme demande le nom du domaine local.

Par exemple :

```text
gestion
```

Il recherche automatiquement le projet correspondant dans :

```text
J:\VirtualHostSlam\gestion\public
```

Si le répertoire existe, le VirtualHost est créé automatiquement.

Le projet devient alors accessible avec :

```text
http://gestion
```

## Pourquoi les droits administrateur sont-ils nécessaires ?

La création ou la suppression d'un VirtualHost nécessite de modifier des fichiers protégés de Windows, notamment :

- la configuration des VirtualHosts Apache ;
- le fichier `hosts` de Windows.

Le programme doit donc être lancé avec les droits administrateur.

## 5.2. Les autres fonctionnalités de `apache.py`

### Lister les VirtualHosts

L'option :

```text
3. Lister les VirtualHosts
```

permet d'afficher les domaines locaux actuellement configurés.

Par exemple :

```text
1. consultation
2. gestion
3. upload
```

### Supprimer un VirtualHost

L'option :

```text
2. Supprimer un VirtualHost
```

permet de sélectionner un VirtualHost existant.

Le programme :

1. demande une confirmation ;
2. supprime la configuration Apache correspondante ;
3. supprime l'entrée correspondante du fichier `hosts` ;
4. redémarre Apache.

**La suppression du VirtualHost ne supprime pas le projet.**

Les fichiers restent présents dans :

```text
J:\VirtualHostSlam
```

### Gestion d'Apache

Le sous-menu :

```text
4. Gestion Apache
```

permet de :

```text
1. Démarrer Apache
2. Arrêter Apache
3. Redémarrer Apache
4. Vérifier le statut
```

# 6. Sauvegarde et restauration des bases MySQL

## Objectif

Les scripts :

```text
save_database.py
restore_database.py
```

permettent respectivement de :

- sauvegarder une ou plusieurs bases MySQL ;
- restaurer une base MySQL à partir d'un fichier `.sql`.

Ces scripts ne nécessitent pas de droits administrateur.

Ils doivent cependant pouvoir créer et utiliser un sous-répertoire `backup`. Ils doivent donc être **copiés dans votre répertoire personnel** et non être exécutés directement depuis le répertoire commun `utilitaire`.

Par exemple :

```text
Votre répertoire personnel
│
└── Sauvegarde
    ├── save_database.py
    ├── restore_database.py
    └── backup
```

Le dossier `backup` est créé automatiquement au même endroit que les scripts.

## Fonctionnement de la sauvegarde

Le programme :

1. recherche les bases utilisateur disponibles ;
2. exclut les bases système ;
3. affiche la liste des bases ;
4. permet d'en sélectionner plusieurs ;
5. réalise un `mysqldump` pour chaque base ;
6. place les fichiers SQL dans le dossier `backup`.

Par exemple :

```text
Votre répertoire personnel
│
└── Sauvegarde
    ├── save_database.py
    ├── restore_database.py
    │
    └── backup
        ├── consultation.sql
        ├── gestion.sql
        └── upload.sql
```

Le script prend également en charge les routines, événements et triggers.

Les informations `DEFINER` sont supprimées des sauvegardes afin de faciliter leur réutilisation sur un autre environnement MySQL.

## Lancement de la sauvegarde

Depuis un terminal placé dans le répertoire contenant le script :

```text
py save_database.py
```

Le script peut également être lancé par double-clic depuis l'Explorateur Windows.

Le programme affiche les bases disponibles et permet de sélectionner celles à sauvegarder.

## Fonctionnement de la restauration

Le script `restore_database.py` :

1. recherche les fichiers SQL présents dans le sous-répertoire `backup` ;
2. affiche la liste numérotée des sauvegardes ;
3. permet d'en sélectionner une ;
4. exécute le fichier SQL afin de restaurer la base.

**Si la base à restaurer existe déjà sur le serveur MySQL, elle est automatiquement supprimée avant la restauration.**

Par exemple :

```text
=== Sauvegardes disponibles ===

1. consultation.sql
2. gestion.sql
3. test.sql
```

Il suffit alors de sélectionner la sauvegarde souhaitée.

## Lancement de la restauration

Depuis un terminal placé dans le répertoire contenant le script :

```text
py restore_database.py
```

Le script peut également être lancé par double-clic depuis l'Explorateur Windows.

# 7. Organisation recommandée du répertoire personnel de l'étudiant

```text
Mon espace personnel
│
└── Sauvegarde
    ├── save_database.py
    ├── restore_database.py
    │
    └── backup
        ├── base1.sql
        ├── base2.sql
        └── ...
```

## Projets PHP

Les projets sont installés sur :

```text
J:\VirtualHostSlam
```

Par exemple :

```text
J:\VirtualHostSlam
│
├── consultation
│   └── public
│
├── gestion
│   └── public
│
└── upload
    └── public
```

# 8. Quel script utiliser ?

Le choix dépend du type de travail demandé.

|Situation|Script à utiliser|
|---|---|
|Projet fourni pour un travail uniquement en salle de TP|`recuperer_td.py`|
|Projet que vous devez pouvoir poursuivre chez vous|`recuperer_projet.py`|
|Créer un domaine local (VirtualHost) pour un projet PHP|`apache.py`|
|Supprimer un domaine local|`apache.py`|
|Gérer Apache|`apache.py`|
|Sauvegarder une base MySQL|`save_database.py`|
|Restaurer une base MySQL|`restore_database.py`|

## À retenir

### Travail uniquement en salle de TP

```text
recuperer_td.py
        ↓
J:\VirtualHostSlam
        ↓
Travail local
        ↓
Récupération éventuelle des modifications du professeur
```

**Aucun dépôt GitHub personnel n'est créé.**

### Projet personnel pouvant être poursuivi chez soi

```text
recuperer_projet.py
        ↓
J:\VirtualHostSlam
        ↓
Votre dépôt GitHub personnel
        ↕
   git push / git pull
        ↕
Travail en salle / travail chez vous
```

Dans les deux cas, les projets sont ensuite exécutés localement avec Apache grâce au VirtualHost créé avec `apache.py`.

Les scripts de base de données sont indépendants de ces deux modes de travail et doivent être conservés dans votre espace personnel.