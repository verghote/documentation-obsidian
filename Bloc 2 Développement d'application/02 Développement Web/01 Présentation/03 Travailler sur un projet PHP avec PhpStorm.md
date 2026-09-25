
Cette documentation décrit la mise en place et l'utilisation quotidienne d'un projet PHP une fois celui-ci récupéré dans :

```text
J:\VirtualHostSlam
```

Elle présente :

- l'ouverture du projet dans PhpStorm ;
- l'installation des dépendances Composer ;
- la configuration du projet pour son exécution avec WampServer ;
- la configuration de la sauvegarde automatique des fichiers ;
- la configuration de la base de données ;
- la sauvegarde et la restauration de la base de données ;
- le travail quotidien avec Git pour les projets disposant d'un dépôt GitHub personnel.

> **Prérequis :** la récupération du projet est décrite dans la documentation [[02 Récupération et gestion des projets avec Git et GitHub]].

# 1. Ouvrir le projet dans PhpStorm

Le projet doit être situé dans :

```text
J:\VirtualHostSlam\nomDuProjet
```

Utiliser l'Explorateur Windows :

```text
Clic droit sur le dossier du projet
→ Open Folder as PhpStorm Project
```

Le projet est maintenant ouvert dans PhpStorm.

# 2. Installation des dépendances Composer

Les projets PHP utilisent **Composer** pour gérer les classes et les composants nécessaires à l'application.

Un fichier :

```text
composer.json
```

est présent à la racine du projet.

Depuis le **Terminal intégré de PhpStorm**, exécuter la commande adaptée au projet.

## Projet utilisant uniquement l'autoload

Lorsque le projet ne nécessite pas l'installation de composants externes et utilise uniquement la section `autoload` du fichier `composer.json` :

```text
composer dump-autoload
```

Cette commande génère notamment :

```text
vendor/autoload.php
```

et permet à PHP de charger automatiquement les classes du projet.

Par exemple :

```text
"autoload": {
    "psr-4": {
        "ClasseTechnique\\": "src/ClasseTechnique/",
        "ClasseMetier\\": "src/ClasseMetier/"
    }
}
```

## Projet utilisant des composants Composer

Lorsque le projet possède des dépendances dans `composer.json`, utiliser :

```text
composer install
```

Cette commande :

- installe les dépendances ;
- crée le répertoire `vendor` ;
- génère l'autoload.

> **Conseil :** `composer install` fonctionne également pour un projet utilisant uniquement l'autoload. Sauf indication contraire du professeur, cette commande peut donc être utilisée dans les deux situations.

Le répertoire :

```text
vendor
```

est créé à la racine du projet.

# 3. Configuration du VirtualHost

Le projet doit être exécuté par Apache en utilisant son répertoire `public` comme racine Web.

La configuration du VirtualHost est réalisée avec l'outil :

```text
utilitaire\apache\apache.py
```

La procédure complète de création, de suppression et de gestion des VirtualHosts est décrite dans la documentation : [[]]

> [[01 Utilitaires]]

Le projet doit notamment respecter la structure :

```text
J:\VirtualHostSlam\nomDuProjet
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

Une fois le VirtualHost créé, le projet est accessible avec une adresse de la forme :

```text
http://nomDuProjet
```

# 4. Sauvegarde automatique du projet avec PhpStorm

Le travail est réalisé directement dans :

```text
J:\VirtualHostSlam\nomDuProjet
```

Cette organisation permet notamment :

- d'exécuter directement le projet avec WampServer ;
- d'utiliser le débogueur PHP de PhpStorm ;
- de travailler dans l'environnement prévu pour les TP.

Cependant, il est recommandé de conserver également une copie des fichiers dans votre espace personnel.

PhpStorm peut réaliser cette copie automatiquement grâce à **Deployment**.

> Cette sauvegarde est une copie de sécurité. Pour les projets possédant un dépôt GitHub personnel, elle ne remplace pas Git et GitHub.

## 4.1. Création du dossier de sauvegarde

Créer dans votre espace personnel un dossier correspondant au projet.

Par exemple :

```text
bloc2/
└── Développement Web/
    └── gestion
```

## 4.2. Configuration de Deployment

Dans PhpStorm :

```text
Tools
→ Deployment
→ Configuration
```

Créer une nouvelle configuration :

```text
Local or mounted folder
```

Choisir le dossier de sauvegarde correspondant au projet.

Par exemple :

```text
bloc2/Développement Web/gestion
```

Le champ **Web server URL** n'est pas nécessaire pour cette utilisation.

### Mapping

Configurer :

```text
Deployment path : .
Web path : .
```

## 4.3. Activer la sauvegarde automatique

Dans PhpStorm :

```text
Tools
→ Deployment
→ Automatic Upload (Always)
```

À partir de ce moment, les modifications effectuées dans le projet peuvent être automatiquement copiées dans votre espace personnel.

Pour effectuer une première sauvegarde complète :

```text
Clic droit sur le projet
→ Upload to gestion
```

# 5. Mise en place de la base de données

Les projets contiennent généralement un répertoire :

```text
sql
```

Celui-ci regroupe les scripts nécessaires à la création et à l'initialisation de la base de données.

Ces scripts peuvent notamment créer :

- la base de données ;
- les tables ;
- les données initiales ;
- les vues ;
- les déclencheurs ;
- les procédures stockées ;
- les utilisateurs et leurs droits.

Les scripts doivent être exécutés **dans l'ordre indiqué par le projet ou par le professeur**.

# 6. Configuration de la base de données dans PhpStorm

Dans PhpStorm, créer une nouvelle source de données :

```text
Database
→ +
→ Data Source
→ MySQL
```

Utiliser les paramètres indiqués pour le projet.

Dans l'environnement de TP, les paramètres sont généralement :

|Paramètre|Valeur|
|---|---|
|SGBD|MySQL|
|Utilisateur|`root`|
|Mot de passe|selon l'environnement fourni|
|Base de données|nom indiqué dans le projet|

Exécuter ensuite les scripts du répertoire `sql` dans l'ordre prévu.

# 7. Sauvegarde et restauration de la base de données

Les scripts :

```text
save_database.py
restore_database.py
```

permettent de sauvegarder et de restaurer les bases MySQL.

Ils doivent être utilisés depuis la copie personnelle placée dans votre espace personnel.

La procédure complète d'installation et d'utilisation de ces scripts est décrite dans la documentation :

> [[01 Utilitaires]]

Les sauvegardes sont conservées dans le répertoire :

```text
backup
```

situé à côté des scripts.

Par exemple :

```text
Mon espace personnel
│
├── save_database.py
├── restore_database.py
│
└── backup
    ├── gestion.sql
    └── consultation.sql
```

# 8. Travail quotidien avec Git et GitHub

Cette partie concerne **uniquement les projets récupérés avec `recuperer_projet.py`**.

Ce script crée votre propre dépôt GitHub à partir du projet fourni par l'enseignant.

Votre dépôt GitHub personnel permet alors de poursuivre le même projet :

- en salle informatique ;
- chez vous ;
- sur plusieurs ordinateurs ;
- au cours de plusieurs séances.

Le principe est le suivant :

```text
                 VOTRE DÉPÔT GITHUB
                       ▲     │
                       │     │
                    push     │ pull
                       │     │
                       │     ▼
              ┌─────────────────────┐
              │ Projet local        │
              │ J:\VirtualHostSlam  │
              └─────────────────────┘
                    ▲          ▲
                    │          │
                 SALLE       DOMICILE
```

GitHub constitue ainsi le **point de synchronisation** entre vos différents environnements de travail.

# 9. Vérifier que le projet utilise votre dépôt GitHub

Dans le **Terminal de PhpStorm**, exécuter :

```bash
git remote -v
```

Pour un projet personnel, le dépôt affiché doit correspondre à **votre dépôt GitHub étudiant**.

Vous ne devez pas retrouver le dépôt GitHub du professeur comme dépôt `origin`.

Cette vérification est particulièrement utile avant votre première utilisation de Git.

# 10. Commencer une séance de travail

Lorsque vous arrivez sur un ordinateur sur lequel vous avez déjà travaillé précédemment, commencez par récupérer la dernière version de votre projet :

```bash
git pull
```

Cette commande récupère les modifications présentes sur GitHub et les intègre dans votre copie locale.

## Pourquoi faut-il faire un `pull` ?

Votre projet peut avoir été modifié :

- lors d'une précédente séance en salle ;
- chez vous ;
- sur un autre ordinateur.

Le `pull` permet donc de travailler sur la version la plus récente disponible sur votre dépôt GitHub.

### À retenir

**Avant de commencer une séance sur un nouvel ordinateur :**

```bash
git pull
```

# 11. Travailler pendant la séance

Vous pouvez ensuite travailler normalement dans PhpStorm :

- modifier les fichiers PHP ;
- créer de nouvelles classes ;
- modifier les fichiers HTML/CSS/JavaScript ;
- modifier les scripts SQL ;
- effectuer vos tests ;
- utiliser le débogueur PHP.

Pendant le travail, vous pouvez vérifier régulièrement l'état du dépôt avec :

```bash
git status
```

Cette commande indique notamment les fichiers modifiés, ajoutés ou supprimés.

# 12. Terminer une séance de travail

Avant de quitter un ordinateur, votre travail doit être enregistré dans Git puis envoyé sur GitHub.

Dans le **Terminal de PhpStorm**, effectuer les commandes suivantes.

## 12.1. Vérifier les modifications

```bash
git status
```

Cela permet de vérifier ce qui a été modifié.

## 12.2. Ajouter les modifications

```bash
git add .
```

Cette commande prépare les modifications pour le prochain commit.

## 12.3. Créer le commit

```bash
git commit -m "Description du travail réalisé"
```

Par exemple :

```bash
git commit -m "Ajout de la gestion des utilisateurs"
```

Le message doit permettre de comprendre rapidement ce qui a été réalisé.

## 12.4. Envoyer le travail sur GitHub

```bash
git push
```

Cette dernière commande est **essentielle**.

Le `commit` enregistre le travail uniquement dans le dépôt Git local de l'ordinateur.

Le `push` envoie ensuite ces modifications sur votre dépôt GitHub.

### Séquence à retenir

À la fin de chaque séance :

```bash
git status
git add .
git commit -m "Description du travail réalisé"
git push
```

> **Ne quittez pas votre poste sans avoir effectué le `push`** si vous souhaitez poursuivre le travail sur un autre ordinateur.

# 13. Travailler chez soi

Le fonctionnement à domicile est exactement le même.

Si vous commencez à travailler chez vous sur un projet qui a déjà été modifié en salle, commencez par :

```bash
git pull
```

Vous récupérez ainsi la dernière version envoyée depuis la salle.

Vous pouvez ensuite travailler normalement.

À la fin de votre séance à domicile :

```bash
git status
git add .
git commit -m "Travail réalisé à domicile"
git push
```

Votre travail est alors disponible sur GitHub.

# 14. Reprendre le travail en salle après avoir travaillé chez soi

Si vous avez travaillé chez vous, il est indispensable d'avoir effectué :

```bash
git push
```

avant de quitter votre ordinateur personnel.

Lorsque vous revenez en salle informatique, ouvrez le projet correspondant puis exécutez :

```bash
git pull
```

Vous récupérez ainsi les modifications réalisées chez vous.

Le cycle est donc :

```text
                 GITHUB ÉTUDIANT
                 /             \
                /               \
           git push           git pull
              /                   \
             ▼                     ▼
      ORDINATEUR SALLE       ORDINATEUR MAISON
             │                     │
             │ travail             │ travail
             │                     │
             └── push ─────────────┘
```

# 15. Exemple complet : salle → domicile → salle

## Fin d'une séance en salle

```bash
git status
git add .
git commit -m "Travail séance 3"
git push
```

Le travail est maintenant sur GitHub.

## Début du travail à domicile

```bash
git pull
```

Le travail réalisé en salle est récupéré.

Vous travaillez ensuite normalement.

## Fin du travail à domicile

```bash
git status
git add .
git commit -m "Travail à domicile"
git push
```

Votre travail est maintenant disponible sur GitHub.

## Retour en salle

```bash
git pull
```

Vous récupérez les modifications réalisées chez vous.

# 16. La règle essentielle

Lorsque vous changez d'ordinateur, retenez simplement :

```text
        JE QUITTE UN ORDINATEUR
                  │
                  ▼
              git add .
                  │
                  ▼
        git commit -m "..."
                  │
                  ▼
              git push
                  │
                  ▼
              GITHUB
                  │
                  ▼
              git pull
                  │
                  ▼
        JE CHANGE D'ORDINATEUR
```

### Règle à retenir

> **Push avant de quitter un ordinateur, pull avant de commencer sur un autre.**

Cette règle permet de synchroniser simplement votre travail entre la salle informatique et votre domicile.

# 17. Les commandes Git indispensables

Pour le travail quotidien, vous devez principalement connaître les commandes suivantes :

|Commande|Rôle|
|---|---|
|`git status`|Affiche l'état du projet|
|`git add .`|Prépare les modifications|
|`git commit -m "..."`|Enregistre les modifications dans l'historique local|
|`git push`|Envoie les commits vers GitHub|
|`git pull`|Récupère les modifications depuis GitHub|
|`git log`|Affiche l'historique des commits|
|`git remote -v`|Affiche les dépôts distants associés au projet|

# 18. Ne pas confondre sauvegarde et Git

Plusieurs mécanismes peuvent être utilisés pour conserver votre travail. Ils ont cependant des rôles différents.

## Sauvegarde PhpStorm / Deployment

Deployment réalise une copie des fichiers dans votre espace personnel :

```text
Projet
   │
   │ Deployment
   ▼
Espace personnel
```

Cette copie constitue une **sauvegarde de sécurité**.

## Git et GitHub

Git conserve l'historique des versions et GitHub permet de synchroniser le projet entre plusieurs ordinateurs :

```text
Projet
   │
   │ git commit
   ▼
Dépôt Git local
   │
   │ git push
   ▼
GitHub
```

Ces deux mécanismes sont donc complémentaires.

# 19. Attention au `push`

Un `commit` seul ne suffit pas lorsque vous changez d'ordinateur.

Par exemple :

```bash
git add .
git commit -m "Ajout de la page d'accueil"
```

Votre travail est bien enregistré, mais uniquement dans le dépôt Git local de l'ordinateur.

Il faut ensuite effectuer :

```bash
git push
```

pour l'envoyer sur GitHub.

Sans `push`, un autre ordinateur ne pourra pas récupérer ce commit avec :

```bash
git pull
```

# 20. Attention au `pull`

De la même manière, avant de commencer à travailler sur un ordinateur qui n'est pas celui sur lequel vous avez terminé votre dernière séance, récupérez votre travail :

```bash
git pull
```

Cela est particulièrement important lorsque vous alternez entre :

```text
Salle informatique
       ↕
     GitHub
       ↕
     Domicile
```

# 21. Résumé

## Tous les projets

Le projet est ouvert dans PhpStorm depuis :

```text
J:\VirtualHostSlam\nomDuProjet
```

Selon les besoins du projet, il faut ensuite configurer :

- Composer ;
- le VirtualHost Apache ;
- la base de données ;
- Deployment.

Les procédures correspondantes sont décrites dans les documentations consacrées aux utilitaires et à la mise en place d'un projet.

## Projet uniquement réalisé en salle

Le projet récupéré avec :

```text
recuperer_td.py
```

reste lié au dépôt GitHub de l'enseignant.

Ce mode est destiné aux projets qui doivent être réalisés uniquement sur les ordinateurs de la salle de TP.

## Projet pouvant être réalisé en salle et à domicile

Le projet récupéré avec :

```text
recuperer_projet.py
```

possède son propre dépôt GitHub étudiant.

Le cycle de travail est alors :

```text
          DÉBUT DE SÉANCE
                │
                ▼
            git pull
                │
                ▼
              TRAVAIL
                │
                ▼
            git status
                │
                ▼
            git add .
                │
                ▼
        git commit -m "..."
                │
                ▼
            git push
                │
                ▼
          FIN DE SÉANCE
```

Et lors d'un changement d'ordinateur :

```text
      ANCIEN ORDINATEUR
             │
          git push
             ▼
           GITHUB
             │
          git pull
             ▼
       NOUVEL ORDINATEUR
```

> **La règle fondamentale :**
> 
> **`git push` avant de quitter un ordinateur.**
> 
> **`git pull` avant de reprendre le travail sur un autre ordinateur.**

Ainsi, votre dépôt GitHub personnel devient le point central de synchronisation de votre projet entre la salle informatique et votre domicile.