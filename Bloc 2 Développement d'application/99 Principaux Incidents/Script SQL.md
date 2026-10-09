
**Requête SQL :** [Copier](http://127.0.0.1/phpmyadmin5.2.3/index.php?route=/import#)

```
/*!40000 DROP DATABASE IF EXISTS `consultation`*/;
```

**MySQL a répondu :** 

`#3681 - Schema 'consultation' does not exist, but schema directory '.\consultation\' was found. This must be resolved manually (e.g. by moving the schema directory to another location).`

Cette erreur provient de MySQL et non directement de phpMyAdmin. Elle apparaît lorsque tu essaies d'importer un fichier SQL qui contient une instruction de suppression de la base de données `consultation`.

## 1. Comprendre l'erreur

Le message :

```
DROP DATABASE IF EXISTS `consultation`;
```

demande à MySQL de supprimer la base de données `consultation` si elle existe.

Or, MySQL renvoie :

```
#3681 - Schema 'consultation' does not exist,
but schema directory '.\consultation\' was found.
This must be resolved manually.
```

Cela signifie que :

- MySQL ne trouve pas la base `consultation` dans ses métadonnées.
- Cependant, il trouve toujours un dossier `consultation` dans son répertoire de données.
- Il existe donc une incohérence entre les informations internes de MySQL et les fichiers présents sur le disque.

En résumé : MySQL pense que la base n'existe pas, mais son dossier physique est toujours présent. Il bloque donc la suppression pour éviter d'aggraver cette incohérence.

## 2. Comment résoudre le problème sous Windows

Précaution

Avant toute manipulation, fais une copie de sauvegarde du dossier de données de MySQL, car une mauvaise manipulation peut entraîner une perte de données.

Étape 1 : Arrêter MySQL

- Ouvre ton panneau de contrôle XAMPP ou WAMP, selon ton installation.
- Arrête le service MySQL.
- Vérifie qu'il est bien arrêté avant de manipuler les fichiers.

Étape 2 : Localiser le dossier de données

Dans notre installation WampServer, il se trouve ici :

```
C:\wamp64\bin\mysql\mysql9.5.0\data
```

Dans ce dossier, tu devrais trouver un répertoire nommé :

```
consultation
```


Étape 3 : Mettre le dossier de côté

- Repèrer le dossier `consultation`.
- Déplacer-le dans un emplacement temporaire situé en dehors du répertoire `data`

Ne pas supprimer ce dossier : il peut contenir des fichiers nécessaires à la récupération de données.

Étape 4 : Redémarrer MySQL

- Redémarrer le service MySQL 
- Retourner dans phpMyAdmin.
- Vérifier que le serveur fonctionne correctement.

Étape 5 : Réimporter ta base de données

1. Ouvrir phpMyAdmin.
2. Sélectionner l'onglet Importer.
3. Choisir ton fichier `.sql`.
4. Lancer l'importation.