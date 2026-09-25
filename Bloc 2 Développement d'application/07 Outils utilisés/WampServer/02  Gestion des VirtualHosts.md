# 1. Objectif

Lorsqu'un développeur travaille sur plusieurs projets Web sur son ordinateur, il est pratique de pouvoir accéder à chaque projet avec une adresse qui lui est propre.

Par exemple :

```text
http://membre.local/
http://catalogue.local/
http://admin.local/
```

plutôt que :

```text
http://localhost/membre/
http://localhost/catalogue/
http://localhost/admin/
```

Pour cela, nous utilisons les **VirtualHosts Apache**.

Cette configuration permet à une seule instance d'Apache de servir plusieurs projets Web, chacun avec son propre nom.

# 2. Qu'est-ce qu'un VirtualHost ?

Un **VirtualHost** permet à Apache de déterminer quelle application doit être servie en fonction du nom utilisé dans l'URL.

Imaginons deux projets :

```text
D:\Projets\membre
D:\Projets\catalogue
```

Nous souhaitons obtenir :

```text
http://membre.local/
http://catalogue.local/
```

Le navigateur envoie une requête à Apache avec le nom demandé.

Apache utilise alors sa configuration des VirtualHosts pour déterminer quel répertoire correspond à ce nom.

```text
http://membre.local/
        │
        ▼
      Apache
        │
        ▼
VirtualHost membre.local
        │
        ▼
D:\Projets\membre
```

et :

```text
http://catalogue.local/
        │
        ▼
      Apache
        │
        ▼
VirtualHost catalogue.local
        │
        ▼
D:\Projets\catalogue
```

Une seule instance d'Apache peut donc servir plusieurs projets.

# 3. Pourquoi utiliser un VirtualHost ?

L'utilisation des VirtualHosts présente plusieurs avantages.

## 3.1 Une adresse par projet

Chaque projet possède sa propre adresse :

```text
membre.local
catalogue.local
admin.local
```

Cela facilite le travail quotidien du développeur.

## 3.2 Une configuration indépendante

Chaque VirtualHost peut avoir sa propre configuration Apache.

On peut notamment définir :

- le répertoire du projet ;
- le nom du site ;
- les règles d'accès ;
- certaines options Apache ;
- les paramètres nécessaires au projet.
    
## 3.3 Des URLs plus proches d'un environnement réel

Avec :

```text
http://membre.local/
```

l'application fonctionne comme un véritable site Web.

Cela peut être particulièrement important pour les applications qui utilisent des URLs absolues, des cookies, des sessions ou certaines règles Apache.

# 4. VirtualHost ou localhost ?

Il faut distinguer deux mécanismes.

### Avec localhost

```text
http://localhost/membre/
```

Apache utilise généralement le répertoire :

```text
C:\wamp64\www\membre
```

### Avec un VirtualHost

```text
http://membre.local/
```

Apache peut utiliser directement :

```text
D:\Projets\membre
```

Le projet n'a donc pas besoin d'être placé dans :

```text
C:\wamp64\www
```

# 5. Comment fonctionne un VirtualHost ?

La configuration repose principalement sur deux éléments :

```text
1. Le fichier hosts de Windows
2. La configuration VirtualHost d'Apache
```

Les deux ont des rôles différents.
# 6. Le fichier hosts de Windows

Windows doit d'abord savoir vers quelle machine correspond le nom :

```text
membre.local
```

Pour cela, on utilise le fichier :

```text
C:\Windows\System32\drivers\etc\hosts
```

On ajoute par exemple :

```text
127.0.0.1 membre.local
```

Cela signifie :

```text
membre.local
     ↓
127.0.0.1
     ↓
ordinateur local
```

Le navigateur sait alors que `membre.local` doit être recherché sur l'ordinateur local.

# 7. La configuration Apache

Le fichier principal utilisé pour les VirtualHosts est :

```text
C:\wamp64\bin\apache\apache2.4.65\conf\extra\httpd-vhosts.conf
```

On y définit le VirtualHost.

Exemple :

```apache
<VirtualHost *:80>

    ServerName membre.local

    DocumentRoot "D:/Projets/membre"

    <Directory "D:/Projets/membre">
        AllowOverride All
        Require all granted
    </Directory>

</VirtualHost>
```

Cette configuration indique à Apache :

> Lorsque le navigateur demande `membre.local`, servir le projet situé dans `D:/Projets/membre`.

# 8. Les deux configurations sont complémentaires

Il est important de comprendre que le fichier `hosts` et Apache ne font pas la même chose.

```text
                  membre.local
                       │
                       ▼
                fichier hosts
                       │
                       ▼
                  127.0.0.1
                       │
                       ▼
                    Apache
                       │
                       ▼
              ServerName membre.local
                       │
                       ▼
             DocumentRoot
                       │
                       ▼
             D:\Projets\membre
```

Le fichier `hosts` permet à Windows de trouver l'ordinateur.

Le VirtualHost permet ensuite à Apache de trouver le bon projet.

# 9. Créer un VirtualHost manuellement

Prenons l'exemple d'un projet :

```text
D:\Projets\membre
```

Nous voulons l'ouvrir avec :

```text
http://membre.local/
```

## Étape 1 — Vérifier le projet

Le répertoire doit exister :

```text
D:\Projets\membre
```

## Étape 2 — Modifier le fichier hosts

Ouvrir en tant qu'administrateur :

```text
C:\Windows\System32\drivers\etc\hosts
```

Ajouter :

```text
127.0.0.1	consultation
::1	consultation
```

## Étape 3 — Ajouter le VirtualHost Apache

Ouvrir :

```text
C:\wamp64\bin\apache\apache2.4.65\conf\extra\httpd-vhosts.conf
```

Ajouter :

```apache
<VirtualHost *:80>
    ServerName consultation
    DocumentRoot "J:/VirtualHostSlam/consultation/public"
    <Directory "J:/VirtualHostSlam/consultation/public/">
        AllowOverride All
        Require local
    </Directory>
</VirtualHost>
```

# 10. Redémarrer Apache

Après une modification de :

```text
httpd-vhosts.conf
```

Apache doit être redémarré pour prendre en compte la nouvelle configuration.

Depuis WampServer :

```text
Clic droit sur l'icône WampServer
    ↓
Apache
    ↓
Service
    ↓
Redémarrer le service
```

# 11. Tester le VirtualHost

Ouvrir dans le navigateur :

```text
http://membre.local/
```

Apache doit alors servir :

```text
D:\Projets\membre
```

Si le projet possède un :

```text
index.php
```

ou :

```text
index.html
```

celui-ci sera normalement exécuté ou affiché.

# 12. Vérifier en cas de problème

Si :

```text
http://membre.local/
```

ne fonctionne pas, vérifier les éléments suivants.

### 1. Le fichier hosts

Vérifier la présence de :

```text
127.0.0.1 membre.local
```

### 2. Le VirtualHost

Vérifier :

```apache
ServerName membre.local
```

### 3. Le DocumentRoot

Vérifier que le répertoire existe réellement :

```text
D:\Projets\membre
```

### 4. Apache

Vérifier que le service Apache fonctionne.

L'icône WampServer doit normalement être verte.

### 5. La configuration Apache

Une erreur dans `httpd-vhosts.conf` peut empêcher Apache de démarrer.

# 13. Vérifier la résolution du nom

Windows permet de vérifier que le nom est correctement associé à l'ordinateur local.

Dans un terminal :

```cmd
ping membre.local
```

Le résultat doit notamment faire apparaître :

```text
127.0.0.1
```

Cela permet de vérifier la partie :

```text
membre.local
     ↓
127.0.0.1
```

Cela ne vérifie cependant pas que le VirtualHost Apache est correctement configuré.

# 14. Créer plusieurs VirtualHosts

Le principe est identique pour chaque projet.

Exemple :

```text
D:\Projets\membre
D:\Projets\catalogue
D:\Projets\admin
```

Dans `hosts` :

```text
127.0.0.1 membre.local
127.0.0.1 catalogue.local
127.0.0.1 admin.local
```

Dans `httpd-vhosts.conf` :

```apache
<VirtualHost *:80>

    ServerName membre.local
    DocumentRoot "D:/Projets/membre"

    <Directory "D:/Projets/membre">
        AllowOverride All
        Require all granted
    </Directory>

</VirtualHost>

<VirtualHost *:80>

    ServerName catalogue.local
    DocumentRoot "D:/Projets/catalogue"

    <Directory "D:/Projets/catalogue">
        AllowOverride All
        Require all granted
    </Directory>

</VirtualHost>

<VirtualHost *:80>

    ServerName admin.local
    DocumentRoot "D:/Projets/admin"

    <Directory "D:/Projets/admin">
        AllowOverride All
        Require all granted
    </Directory>

</VirtualHost>
```

On peut alors accéder aux trois applications avec :

```text
http://membre.local/
http://catalogue.local/
http://admin.local/
```

# 15. Modifier un VirtualHost

Lorsqu'un projet change de répertoire, il faut modifier le `DocumentRoot`.

Par exemple :

```text
D:\Projets\membre
```

devient :

```text
D:\Developpement\membre
```

Il faut modifier :

```apache
DocumentRoot "D:/Developpement/membre"

<Directory "D:/Developpement/membre">
    AllowOverride All
    Require all granted
</Directory>
```

Puis redémarrer Apache.

# 16. Supprimer un VirtualHost

La suppression d'un VirtualHost nécessite de modifier **deux configurations** :

```text
1. La configuration Apache
2. Le fichier hosts
```

Par exemple, pour supprimer :

```text
membre.local
```

il faut supprimer le bloc correspondant dans :

```text
httpd-vhosts.conf
```

et supprimer l'entrée correspondante dans :

```text
hosts
```

c'est-à-dire :

```text
127.0.0.1 membre.local
```

Puis redémarrer Apache.

# 17. Attention lors de la suppression d'un domaine

La suppression d'un domaine doit être effectuée avec une correspondance **exacte**.

Par exemple, si le fichier `hosts` contient :

```text
127.0.0.1 vds.local
127.0.0.1 vds-correction.local
```

la suppression de :

```text
vds.local
```

ne doit supprimer que :

```text
127.0.0.1 vds.local
```

et surtout pas :

```text
127.0.0.1 vds-correction.local
```

Une recherche trop simple basée uniquement sur :

```text
vds
```

pourrait supprimer plusieurs entrées.

C'est pourquoi un outil d'automatisation doit vérifier le **nom de domaine complet** et non rechercher simplement une partie du texte.

# 18. Automatiser la gestion avec un utilitaire Python

La création et la suppression manuelle de VirtualHosts sont relativement simples, mais elles nécessitent de modifier plusieurs fichiers avec des droits administrateur.

Pour éviter les erreurs et accélérer les opérations courantes, un utilitaire Python peut automatiser ces tâches.

L'objectif est notamment de pouvoir :

```text
Créer un VirtualHost
Modifier un VirtualHost
Supprimer un VirtualHost
Redémarrer Apache
```

depuis un menu unique.

L'utilitaire doit notamment gérer :

```text
httpd-vhosts.conf
hosts
service Apache
```

# 19. Pourquoi automatiser la suppression ?

Une suppression manuelle peut facilement laisser une configuration incomplète.

Par exemple :

```text
VirtualHost supprimé
        +
entrée hosts conservée
```

ou l'inverse :

```text
entrée hosts supprimée
        +
VirtualHost conservé
```

L'application devient alors difficile à comprendre.

L'utilitaire permet d'effectuer les deux opérations ensemble :

```text
Suppression de membre.local
        │
        ├──► suppression du VirtualHost Apache
        │
        ├──► suppression de l'entrée hosts
        │
        └──► redémarrage d'Apache
```

# 20. Principe du menu de gestion

L'utilitaire peut proposer un menu tel que :

```text
====================================
       GESTION DES VIRTUALHOSTS
====================================

1. Lister les VirtualHosts
2. Créer un VirtualHost
3. Modifier un VirtualHost
4. Supprimer un VirtualHost
5. Redémarrer Apache
6. Quitter
```

Le développeur n'a alors plus besoin de modifier directement les fichiers de configuration pour les opérations courantes.

# 21. Droits administrateur

La modification du fichier :

```text
C:\Windows\System32\drivers\etc\hosts
```

nécessite généralement des droits administrateur.

Le programme de gestion des VirtualHosts doit donc être exécuté avec les privilèges administrateur.

Si ce n'est pas le cas, l'utilitaire doit afficher un message explicite :

```text
Exécuter l'utilitaire en tant qu'administrateur.
```

# 22. Vérification après création

Après avoir créé un VirtualHost, effectuer les vérifications suivantes :

### Vérification 1

Le domaine existe dans :

```text
hosts
```

Exemple :

```text
127.0.0.1 membre.local
```

### Vérification 2

Le VirtualHost existe dans :

```text
httpd-vhosts.conf
```

### Vérification 3

Apache a été redémarré.

### Vérification 4

Le domaine répond :

```text
http://membre.local/
```

# 23. Vérification après suppression

Après avoir supprimé un VirtualHost :

### Vérification 1

L'entrée n'existe plus dans :

```text
hosts
```

### Vérification 2

Le bloc `<VirtualHost>` correspondant n'existe plus dans :

```text
httpd-vhosts.conf
```

### Vérification 3

Apache a été redémarré.

Le domaine ne doit alors plus être utilisé pour accéder au projet.

# 24. Schéma général

La configuration complète d'un projet peut être résumée ainsi :

```text
                     Navigateur
                         │
                         │
              http://membre.local/
                         │
                         ▼
                  Fichier hosts
                         │
                         │
                    127.0.0.1
                         │
                         ▼
                      Apache
                         │
                         │ ServerName
                         │ membre.local
                         ▼
                   VirtualHost
                         │
                         │ DocumentRoot
                         ▼
                  D:\Projets\membre
                         │
                         ▼
                    Application
                         │
                         ▼
                       PHP
```

# 25. À retenir

Pour qu'un VirtualHost fonctionne, trois éléments doivent être cohérents :

```text
        Domaine
          │
          ▼
        hosts
          │
          ▼
     127.0.0.1
          │
          ▼
       Apache
          │
          ▼
   ServerName
          │
          ▼
    DocumentRoot
          │
          ▼
       Projet
```

Par exemple :

```text
Domaine :
membre.local

hosts :
127.0.0.1 membre.local

ServerName :
membre.local

DocumentRoot :
D:/Projets/membre
```

Ces quatre informations doivent correspondre.

# 26. Procédure rapide

Pour créer un nouveau projet :

```text
1. Créer ou récupérer le projet
          ↓
2. Choisir son nom de domaine local
          ↓
3. Ajouter le domaine dans hosts
          ↓
4. Créer le VirtualHost Apache
          ↓
5. Redémarrer Apache
          ↓
6. Ouvrir le domaine dans le navigateur
```

Exemple :

```text
Projet :
D:\Projets\monprojet

Domaine :
monprojet.local

hosts :
127.0.0.1 monprojet.local

VirtualHost :
ServerName monprojet.local
DocumentRoot "D:/Projets/monprojet"

URL :
http://monprojet.local/
```

Pour supprimer un projet :

```text
1. Supprimer le VirtualHost
          ↓
2. Supprimer l'entrée hosts
          ↓
3. Redémarrer Apache
          ↓
4. Vérifier
```

# 27. Documentation associée

Cette documentation décrit le fonctionnement des VirtualHosts.

Pour automatiser leur gestion, utiliser l'utilitaire :

```text
virtual_host.py
```

La documentation Xdebug et PhpStorm est présentée séparément dans :

**03 - PHP - Xdebug et PhpStorm**

Le développeur dispose ainsi de trois niveaux indépendants :

```text
01 - WampServer
      │
      ▼
02 - VirtualHosts
      │
      ▼
03 - Xdebug + PhpStorm
```