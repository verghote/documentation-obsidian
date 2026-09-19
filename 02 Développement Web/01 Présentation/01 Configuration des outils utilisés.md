
Chaque poste dispose des outils suivants :

- Le logiciel **Git** accessible depuis l'interpréteur de commande `cmd`.
- L'utilitaire **Cmder** comprenant sa propre version de Git.
- L’interface en ligne de commande officielle de GitHub : **gh**.
- L'IDE **PhpStorm**.
- L'IDE **DataGrip**.
- Le client **MySQL Workbench**.
- La plateforme de développement Web **WampServer**.
- **Composer**, le gestionnaire de dépendances officiel de PHP.
- **Postman**, un outil de test et de développement d’API.
- **JsHint**, un outil intégré dans PhpStorm permettant d'analyse le code Javascript à la recherche d'erreur 

Certains outils nécessitent d'être configurés afin de pouvoir les utiliser

# 1. Configuration initiale de Git

Avant d'utiliser Git, configurez les paramètres suivants dans un terminal :

- définir votre nom, qui sera associé aux commits ;
- définir votre adresse e-mail, qui sera associée aux commits ;
- définir `main` comme nom de branche par défaut lors de la création d'un dépôt ;
- activer la conversion automatique des fins de ligne sous Windows ;
- définir **Notepad++** comme éditeur de texte par défaut pour Git.

Cette configuration peut être réalisée **globalement**, pour l'ensemble de vos projets Git, ou **localement**, uniquement pour un projet donné.

> **Attention — Salle 27 :** en salle 27, la configuration globale de Git n'est pas possible en raison de la configuration des comptes et des répertoires réseau. Dans cette salle, les paramètres Git doivent donc être configurés **localement dans chaque projet**.

## Configuration globale

Sur un ordinateur personnel ou dans un environnement permettant la configuration globale, utilisez :

```text
git config --global user.name "Votre Nom"
git config --global user.email "votre.email@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

Vous pouvez vérifier la configuration avec :

```text
git config --global --list
```

## Configuration locale d'un projet

En **salle 27**, placez-vous dans le répertoire du projet avant d'exécuter les commandes.

Par exemple :

```text
cd J:\VirtualHostSlam\mon-projet
```

Puis configurez Git uniquement pour ce projet :

```text
git config user.name "Votre Nom"
git config user.email "votre.email@example.com"
git config init.defaultBranch main
git config core.autocrlf true
git config core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

Notez l'usage de l'apostrophe pour délimiter le chemin d'accès vers notepad++ cat ce chemin contient des espaces.
On peut aussi utiliser les guillemets en les échappant :
```
git config --local core.editor "\"C:/Program Files/Notepad++/notepad++.exe\" -multiInst -notabbar -nosession -noPlugin"
```

Notepad++ sera utilisé si le dévelopeur lance la command git commit sans donner une description avec l'option -m "description du commit"

Exemple 

```
git commit
```

Notepad++ s'ouvre : vous permettant de saisir la description après le dernier #
``` 

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch main
# Your branch is up to date with 'origin/main'.
#
# Changes to be committed:
#	new file:   test.txt
#
```

La configuration est alors enregistrée dans le fichier :

```text
J:\VirtualHostSlam\mon-projet\.git\config
```

Elle ne s'applique qu'à ce dépôt.

Vous pouvez vérifier la configuration locale avec :

```text
git config --local --list
```

ou simplement :

```text
git config --list
```

## Quelle configuration utiliser ?

|Environnement|Configuration|
|---|---|
|Ordinateur personnel|Globale recommandée|
|Salle 27|Locale obligatoire|
|Projet nécessitant une configuration particulière|Locale|

La configuration locale **prend priorité sur la configuration globale** lorsqu'une même option est définie dans les deux niveaux.

Ainsi, il est possible d'avoir une configuration globale sur votre ordinateur personnel tout en utilisant des paramètres différents pour un projet particulier.
# 2. Configuration initiale de l'interface en ligne de commande GitHub (`gh`)

L'utilisation des commandes `gh` nécessite une phase initiale d'authentification permettant l'accès à votre compte GitHub.
### Étapes

1. Ouvrir une fenêtre **Cmder**.
2. Exécuter la commande suivante :

```
gh auth login --web --scopes delete_repo
```

3. Le navigateur Web s'ouvre automatiquement.
4. Se connecter à son compte GitHub.
5. Recopier le code affiché dans la page d'authentification lorsque cela est demandé.

Pour plus de détails sur la commande gh :  [[Interface en ligne de commande gh.pdf]]

# 3 Première connexion à PhpStorm

Lors de la première utilisation de **PhpStorm**, il est nécessaire d'activer l'IDE à l'aide de la licence serveur mise à disposition par le lycée.
## Prérequis

- Disposer d'un compte **JetBrains**.
- Utiliser l'adresse électronique du lycée : prenom.nom@saint-remi.net
## Activation de la licence

1. Lancer **PhpStorm**.
2. Si une fenêtre d'activation apparaît, choisir : License Server
3. Saisir l'adresse du serveur de licences : https://saintremi.fls.jetbrains.com
4. Valider.

PhpStorm s'active automatiquement si votre compte est autorisé.
## Vérifier l'activation : Menu Help → Register...

La fenêtre doit indiquer que la licence est active.

> **Remarque**
> 
> Si l'activation échoue, vérifiez que vous utilisez bien votre compte JetBrains associé à votre adresse électronique du lycée (`prenom.nom@saint-remi.net`). En cas de difficulté, contactez votre enseignant.

[[Accéder à PhpStorm]]

# 4. Configuration de JsHint

[[Analyse du code avec JSHint]]