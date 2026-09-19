
**WampServer** est une plateforme de développement Web destinée à Windows.

Le nom WAMP correspond aux principaux composants de l'environnement :

- **W** : Windows
- **A** : Apache
- **M** : MySQL / MariaDB
- **P** : PHP

WampServer permet de disposer sur un même ordinateur de l'ensemble des composants nécessaires au développement et à l'exécution d'applications Web PHP.

L'environnement comprend notamment :

- **Apache** : serveur Web ;
- **PHP** : langage utilisé par les applications Web ;
- **MySQL** : système de gestion de bases de données ;
- **MariaDB** : autre système de gestion de bases de données compatible avec de nombreux usages MySQL ;
- **phpMyAdmin** : application Web permettant d'administrer les bases de données.
    
Dans notre environnement, WampServer est installé dans :

```text
C:\wamp64
```

# 2. Comment fonctionne WampServer ?

Lorsqu'un développeur ouvre une application Web dans son navigateur, plusieurs composants interviennent.

Par exemple :

```text
http://monprojet.local/
```

Le navigateur envoie une requête à l'ordinateur local.

Apache reçoit cette requête et détermine quelle application doit être exécutée.

Si la page demandée contient du PHP, Apache fait intervenir PHP.

L'application PHP peut ensuite communiquer avec MySQL ou MariaDB pour récupérer ou enregistrer des données.

Le résultat est finalement retourné au navigateur.

Le fonctionnement peut être représenté ainsi :

```text
Navigateur
     │
     │ HTTP
     ▼
  Apache
     │
     ▼
    PHP
     │
     ├──────────────► MySQL
     │
     └──────────────► MariaDB
     │
     ▼
Résultat HTML
     │
     ▼
Navigateur
```

Le développeur travaille principalement sur le code de l'application. WampServer fournit l'environnement nécessaire pour exécuter ce code localement.

# 3. Apache

**Apache** est le serveur Web de WampServer.

Son rôle est de recevoir les requêtes HTTP provenant du navigateur et de fournir les ressources demandées.

Par exemple, lorsque le navigateur demande :

```text
http://localhost/
```

Apache reçoit la requête.

Lorsqu'il s'agit d'une page PHP, Apache permet son exécution par PHP avant de retourner le résultat au navigateur.

Apache fonctionne sous la forme d'un **service Windows**.

Dans notre environnement, les fichiers d'Apache se trouvent notamment dans :

```text
C:\wamp64\bin\apache\
```

La version actuellement utilisée peut être consultée depuis WampServer.

# 4. PHP

**PHP** est le langage de programmation utilisé par les applications Web.

Le code PHP est exécuté sur le serveur et non dans le navigateur.

Par exemple :

```php
<?php

echo "Bonjour";
```

Le navigateur ne reçoit pas le code PHP.

Il reçoit le résultat :

```text
Bonjour
```

Plusieurs versions de PHP peuvent être installées simultanément dans WampServer.

Dans notre environnement, plusieurs versions de PHP sont disponibles, notamment dans la famille :

```text
PHP 8.0.x à PHP 8.5.x
```

La version réellement utilisée dépend de la configuration sélectionnée dans WampServer.

Le développeur doit donc vérifier la version de PHP utilisée par son projet lorsque celui-ci impose une version particulière.

# 5. MySQL et MariaDB

## 5.1 MySQL

**MySQL** est un système de gestion de bases de données relationnelles.

Une application PHP peut utiliser MySQL pour stocker les données nécessaires à son fonctionnement :

- utilisateurs ;
- produits ;
- commandes ;
- paramètres ;
- etc.
    

Dans notre environnement :

```text
MySQL 9.5.0
```

MySQL fonctionne sous la forme d'un **service Windows**.

## 5.2 MariaDB

**MariaDB** est un système de gestion de bases de données issu d'un fork de MySQL.

Il est compatible avec de nombreux outils et applications conçus pour MySQL.

Dans notre environnement :

```text
MariaDB 11.4.9
```

MySQL et MariaDB sont deux systèmes de bases de données distincts.

Ils peuvent être installés simultanément dans WampServer, mais il faut faire attention aux éventuels conflits de ports lorsqu'ils sont démarrés simultanément.

# 6. phpMyAdmin

**phpMyAdmin** est une application Web permettant d'administrer les bases de données MySQL et MariaDB.

Il permet notamment de :

- créer une base de données ;
- supprimer une base de données ;
- créer et modifier des tables ;
- consulter les données ;
- exécuter des requêtes SQL ;
- importer une base de données ;
- exporter une base de données.

L'accès se fait généralement depuis :

```text
http://localhost/phpmyadmin/
```

phpMyAdmin est particulièrement pratique pour effectuer les opérations courantes sur les bases pendant le développement.

# 7. Les services Windows

Apache, MySQL et MariaDB fonctionnent sous la forme de **services Windows**.

Un service Windows peut démarrer automatiquement avec Windows.

Dans notre environnement, les services nécessaires sont normalement configurés pour démarrer automatiquement.

Cela signifie que le serveur Web peut être disponible même si l'interface graphique de WampServer n'est pas ouverte.

Il faut donc distinguer deux éléments :
### Les services

Ils font réellement fonctionner :

```text
Apache
MySQL
MariaDB
```

### Le gestionnaire WampServer

Le programme :

```text
C:\wamp64\wampmanager.exe
```

fournit une interface permettant d'administrer facilement ces composants.

# 8. L'icône WampServer

L'icône WampServer peut être affichée dans la zone de notification de Windows.

Elle permet d'accéder rapidement aux fonctions d'administration de WampServer.

Un clic gauche ou un clic droit sur cette icône permet notamment d'accéder :

- aux services ;
- aux versions d'Apache ;
- aux versions de PHP ;
- aux versions de MySQL/MariaDB ;
- aux fichiers de configuration ;
- aux outils WampServer.
    
# 9. Signification de la couleur de l'icône

La couleur de l'icône indique l'état général des services.

## 🟢 Vert

Les services nécessaires fonctionnent correctement.

L'environnement est normalement opérationnel.

## 🟠 Orange

Au moins un service n'a pas démarré correctement.

Il faut identifier le service concerné.

Par exemple :

```text
Apache
```

ou :

```text
MySQL
```

## 🔴 Rouge

Les services nécessaires ne fonctionnent pas correctement.

L'environnement n'est généralement pas opérationnel.

# 10. Vérifier que WampServer fonctionne

Avant de rechercher un problème dans une application PHP, il faut vérifier que WampServer fonctionne correctement.

La première vérification consiste à regarder la couleur de l'icône WampServer.

Elle doit être :

```text
🟢 Verte
```

On peut ensuite ouvrir dans le navigateur :

```text
http://localhost/
```

Si la page d'accueil WampServer apparaît, Apache répond correctement aux requêtes HTTP.

# 11. Le répertoire principal de WampServer

WampServer est installé dans :

```text
C:\wamp64
```

Les principaux répertoires sont :

```text
C:\wamp64
│
├── alias
├── apps
├── bin
├── data
├── www
└── ...
```

Les répertoires les plus importants pour le développeur sont présentés ci-dessous.

# 12. Le répertoire www

Le répertoire :

```text
C:\wamp64\www
```

est le répertoire Web principal de WampServer.

Il peut contenir les applications Web utilisées pour le développement.

Par exemple :

```text
C:\wamp64\www\monprojet
```

peut être accessible par :

```text
http://localhost/monprojet/
```

Le fichier :

```text
C:\wamp64\www\index.php
```

correspond à la page d'accueil utilisée par défaut pour :

```text
http://localhost/
```

Dans notre environnement, les projets peuvent également être placés en dehors de `www` et être accessibles grâce aux **VirtualHosts**.

La gestion des VirtualHosts fait l'objet d'une documentation séparée.

# 13. Le répertoire bin

Le répertoire :

```text
C:\wamp64\bin
```

contient les composants exécutables de WampServer.

On y trouve notamment :

```text
C:\wamp64\bin\apache\
C:\wamp64\bin\php\
C:\wamp64\bin\mysql\
C:\wamp64\bin\mariadb\
```

Plusieurs versions d'un même composant peuvent être installées.

Par exemple :

```text
C:\wamp64\bin\php\
    ├── php8.0.x
    ├── php8.1.x
    ├── php8.2.x
    ├── php8.3.28
    └── ...
```

WampServer permet de sélectionner la version utilisée.

# 14. Le répertoire apps

Le répertoire :

```text
C:\wamp64\apps
```

contient différentes applications fournies avec WampServer.

On y trouve notamment :

```text
phpMyAdmin
```

Ces applications permettent d'administrer ou d'utiliser plus facilement l'environnement de développement.

# 15. Le répertoire data

Le répertoire :

```text
C:\wamp64\data
```

contient les données utilisées par les serveurs de bases de données.

**Il ne faut pas modifier directement les fichiers contenus dans ce répertoire.**

Pour manipuler les bases de données, utiliser :

- phpMyAdmin ;
    
- un client MySQL/MariaDB ;
    
- ou les outils spécifiques utilisés par le projet.
    
# 16. Le répertoire alias

Le répertoire :

```text
C:\wamp64\alias
```

contient des fichiers de configuration Apache permettant de définir des **alias**.

Un alias permet de faire correspondre une URL à un répertoire du disque.

Par exemple :

```text
http://localhost/membre/
```

peut pointer vers :

```text
D:\Projets\membre
```

L'alias permet donc d'utiliser un répertoire situé en dehors de :

```text
C:\wamp64\www
```

Cependant, pour nos projets Web, nous privilégions l'utilisation des **VirtualHosts**.

La gestion des VirtualHosts est décrite dans la documentation :

**02 - WampServer - Gestion des VirtualHosts**

# 17. Les principaux fichiers de configuration

## 17.1 php.ini

Le fichier `php.ini` contient la configuration de PHP.

Il permet notamment de configurer :

- les extensions PHP ;
- la mémoire disponible ;
- la durée maximale d'exécution ;
- la taille maximale des fichiers envoyés ;
- l'affichage des erreurs ;
- Xdebug ;
- etc.

Dans notre environnement Apache, le fichier utilisé peut notamment se trouver ici :

```text
C:\wamp64\bin\apache\apache2.4.65\bin\php.ini
```

**Attention :** lorsqu'il existe plusieurs versions de PHP, le fichier `php.ini` utilisé dépend de la version active.

Après une modification du `php.ini`, il faut généralement redémarrer Apache pour que la modification soit prise en compte.

# 18. httpd.conf

`httpd.conf` est le fichier de configuration principal d'Apache.

Il permet notamment de configurer :

- le port d'écoute d'Apache ;
- les modules Apache ;
- les répertoires ;
- les règles d'accès ;
- les fichiers de configuration complémentaires ;
- les VirtualHosts.

Il se trouve notamment sous :

```text
C:\wamp64\bin\apache\apache2.4.65\conf\
```

# 19. httpd-vhosts.conf

Les VirtualHosts sont généralement configurés dans :

```text
C:\wamp64\bin\apache\apache2.4.65\conf\extra\httpd-vhosts.conf
```

Ce fichier permet de déclarer les différents sites Web locaux.

Par exemple :

```apache
<VirtualHost *:80>

    ServerName monprojet.local

    DocumentRoot "D:/Projets/monprojet"

    <Directory "D:/Projets/monprojet">
        AllowOverride All
        Require all granted
    </Directory>

</VirtualHost>
```

La configuration détaillée des VirtualHosts est décrite dans le document : **02 Gestion des VirtualHosts**

# 20. Démarrer, arrêter ou redémarrer Apache

Il peut être nécessaire de redémarrer Apache après une modification de configuration.

Depuis l'icône WampServer :

```text
Clic droit
    ↓
Apache
    ↓
Service
    ↓
Redémarrer le service
```

Il est également possible d'effectuer ces opérations depuis les outils d'administration proposés par WampServer.

# 21. Quand faut-il redémarrer Apache ?

Un redémarrage d'Apache est généralement nécessaire après une modification de :

```text
php.ini
```

ou :

```text
httpd.conf
```

ou :

```text
httpd-vhosts.conf
```

Il est également nécessaire après certaines modifications concernant les extensions PHP.

En revanche, la modification du code PHP d'une application ne nécessite normalement pas de redémarrer Apache.

Par exemple, après avoir modifié :

```text
index.php
```

il suffit généralement de recharger la page dans le navigateur.

# 22. Que faire si localhost ne fonctionne pas ?

Si :

```text
http://localhost/
```

ne fonctionne pas, vérifier dans l'ordre :

### 1. Vérifier l'icône WampServer

Elle doit être verte.

### 2. Vérifier Apache

S'assurer que le service Apache est démarré.

### 3. Vérifier le port

Apache utilise généralement le port :

```text
80
```

Un autre logiciel peut déjà utiliser ce port.

### 4. Vérifier la configuration Apache

Une erreur dans `httpd.conf` ou dans un fichier de VirtualHost peut empêcher Apache de démarrer.

### 5. Consulter les journaux Apache

Les fichiers de log Apache peuvent fournir des informations permettant d'identifier l'erreur.

# 23. WampServer et les projets de développement

Dans notre environnement, chaque projet Web peut être configuré avec son propre **VirtualHost**.

Par exemple :

```text
Projet 1
D:\Projets\membre
       ↓
membre.local

Projet 2
D:\Projets\catalogue
       ↓
catalogue.local

Projet 3
D:\Projets\admin
       ↓
admin.local
```

Le développeur accède alors aux projets directement avec leur nom :

```text
http://membre.local/
http://catalogue.local/
http://admin.local/
```

Cette organisation est préférable à l'utilisation systématique d'URL de type :

```text
http://localhost/monprojet/
```

La création, la modification et la suppression des VirtualHosts font l'objet du document : **02 Gestion des VirtualHosts**

# 24. WampServer et PhpStorm

WampServer et PhpStorm ont des rôles différents.

### WampServer

Fournit l'environnement d'exécution :

```text
Apache
PHP
MySQL / MariaDB
```

### PhpStorm

Permet au développeur de :

- modifier le code ;
- gérer le projet ;
- effectuer des recherches ;
- utiliser Git ;
- lancer des outils ;
- déboguer le code PHP.
    

Pour le débogage PHP, Xdebug fait le lien entre l'application exécutée par WampServer et PhpStorm.

```text
Navigateur
     ↓
Apache
     ↓
PHP + Xdebug
     ↓
PhpStorm
```

La configuration de Xdebug et PhpStorm est décrite dans : **03 - PHP - Xdebug et PhpStorm**

# 25. Résumé

Les principaux éléments à retenir sont :

|Élément|Rôle|
|---|---|
|Apache|Serveur Web|
|PHP|Exécution du code PHP|
|MySQL|Base de données|
|MariaDB|Base de données|
|phpMyAdmin|Administration des bases|
|`www`|Répertoire Web principal|
|`bin`|Composants et différentes versions|
|`alias`|Configuration des alias Apache|
|`data`|Données des bases|
|`php.ini`|Configuration de PHP|
|`httpd.conf`|Configuration principale d'Apache|
|`httpd-vhosts.conf`|Configuration des VirtualHosts|

Le fonctionnement général est :

```text
                   WampServer
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     Apache           PHP       MySQL / MariaDB
        │              │
        └───────┬──────┘
                ↓
          Application Web
                │
                ↓
            Navigateur
```

Pour travailler sur un projet :

```text
1. WampServer opérationnel
           ↓
2. VirtualHost configuré
           ↓
3. Projet accessible dans le navigateur
           ↓
4. Développement dans PhpStorm
           ↓
5. Xdebug si le débogage est nécessaire
```

Les procédures spécifiques sont documentées séparément :

**02 - WampServer - Gestion des VirtualHosts**

**03 - PHP - Xdebug et PhpStorm**