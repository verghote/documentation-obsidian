# 1. Objectif

Cette documentation explique comment mettre en place le débogage PHP avec :

- **WampServer** ;
- **PHP** ;
- **Xdebug** ;
- **PhpStorm**.
    

L'objectif est de pouvoir exécuter une application PHP et, lorsque cela est nécessaire, interrompre son exécution dans PhpStorm afin d'inspecter le code.

Le développeur pourra notamment :

- placer des points d'arrêt ;
- exécuter le code pas à pas ;
- consulter les variables ;
- examiner la pile d'appels ;
- comprendre pourquoi une partie du code ne produit pas le résultat attendu.

Cette documentation suppose que :

1. WampServer fonctionne correctement ;
2. le projet PHP est accessible depuis le navigateur ;
3. le projet utilise un VirtualHost lorsque cela est nécessaire.

La configuration des VirtualHosts est décrite dans : **02 Gestion des VirtualHosts**

# 2. Pourquoi utiliser Xdebug ?

Lorsqu'une application PHP rencontre un problème, ajouter des `echo`, `var_dump()` ou `print_r()` peut être utile.

Cependant, cette méthode devient rapidement difficile à utiliser lorsque le problème se situe dans un traitement complexe.

Xdebug permet au contraire de **suspendre l'exécution du programme à un endroit précis**.

Par exemple :

```php
$user = getUser($id);

$name = $user['name'];

echo $name;
```

On peut placer un point d'arrêt sur :

```php
$name = $user['name'];
```

Lorsque cette ligne est atteinte, PhpStorm interrompt l'exécution.

Le développeur peut alors examiner :

```text
$id
$user
$user['name']
```

Il peut également poursuivre l'exécution ligne par ligne.

# 3. Comment fonctionne le débogage ?

Il est important de comprendre que Xdebug et PhpStorm ont deux rôles différents.

**Xdebug** est installé avec PHP et intervient pendant l'exécution du code.

**PhpStorm** reçoit la connexion de Xdebug et fournit l'interface permettant de contrôler le débogage.

Le fonctionnement est le suivant :

```text
Navigateur
     │
     │ requête HTTP
     ▼
   Apache
     │
     ▼
    PHP
     │
     ▼
  Xdebug
     │
     │ connexion de débogage
     ▼
 PhpStorm
     │
     ▼
Breakpoint
```

PhpStorm n'exécute donc pas le code PHP à la place de WampServer.

Le code est exécuté par PHP sous Apache.

Xdebug permet à cette exécution de communiquer avec PhpStorm.
# 4. Vérifier que Xdebug est installé

Dans WampServer :

```text
Clic gauche sur l'icône WampServer
    ↓
PHP
    ↓
Extensions PHP
```

Vérifier que l'extension :

```text
php_xdebug
```

est activée.

Une autre méthode consiste à utiliser `phpinfo()`.

Créer temporairement un fichier :

```text
phpinfo.php
```

avec :

```php
<?php

phpinfo();
```

Puis ouvrir :

```text
http://localhost/phpinfo.php
```

Rechercher :

```text
Xdebug
```

Si une section Xdebug apparaît, l'extension est chargée par PHP.

Après vérification, le fichier `phpinfo.php` peut être supprimé.

# 5. Configurer Xdebug dans php.ini

Le fichier `php.ini` utilisé par Apache se trouve dans notre environnement ici :

```text
C:\wamp64\bin\apache\apache2.4.65\bin\php.ini
```

La configuration actuelle de Xdebug est la suivante :

```ini
[xdebug]

zend_extension="c:/wamp64/bin/php/php8.3.28/zend_ext/php_xdebug-3.4.7-8.3-ts-vs16-x86_64.dll"

xdebug.mode=develop,debug
xdebug.start_with_request=yes

xdebug.client_host=127.0.0.1
xdebug.client_port=9003

xdebug.output_dir="c:/wamp64/tmp"

xdebug.log="c:/wamp64/logs/xdebug.log"
xdebug.log_level=7

xdebug.show_local_vars=0
xdebug.profiler_output_name=trace.%H.%t.%p.cgrind
xdebug.use_compression=false
```

Les paramètres importants pour le débogage sont :

```ini
xdebug.mode=develop,debug
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```
# 6. Comprendre les paramètres Xdebug

## 6.1 xdebug.mode

```ini
xdebug.mode=develop,debug
```

Le mode `develop` fournit des fonctionnalités utiles au développement.

Le mode `debug` active le débogage pas à pas avec un client tel que PhpStorm.

## 6.2 xdebug.start_with_request

```ini
xdebug.start_with_request=yes
```

Ce paramètre est important.

Avec cette configuration, **Xdebug démarre une session de débogage pour les requêtes PHP**.

C'est la configuration la plus simple pour commencer.

Le développeur n'a pas besoin d'ajouter manuellement un trigger dans l'URL.

En contrepartie, lorsque PhpStorm est en mode écoute, les requêtes PHP peuvent être interceptées par le débogueur.

C'est ce comportement qui explique notamment pourquoi il faut savoir **arrêter l'écoute de PhpStorm lorsqu'on ne souhaite plus déboguer**.

Cette notion est détaillée dans la section consacrée au démarrage et à l'arrêt d'une session.

## 6.3 xdebug.client_host

```ini
xdebug.client_host=127.0.0.1
```

Cette adresse indique à Xdebug où trouver le programme qui reçoit les connexions de débogage.

Dans notre environnement, PhpStorm fonctionne sur le même ordinateur que WampServer.

Nous utilisons donc :

```text
127.0.0.1
```

qui correspond à l'ordinateur local.

## 6.4 xdebug.client_port

```ini
xdebug.client_port=9003
```

Le port `9003` est le port utilisé par défaut par Xdebug 3 pour les connexions de débogage.

PhpStorm doit utiliser le même port.

# 7. Redémarrer Apache

Après avoir modifié `php.ini`, Apache doit être redémarré.

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

Cette étape est importante : une modification du fichier `php.ini` ne modifie pas automatiquement la configuration du processus Apache déjà en fonctionnement.

# 8. Vérifier la configuration de Xdebug

Retourner sur :

```text
http://localhost/phpinfo.php
```

Rechercher la section Xdebug.

Vérifier notamment :

```text
xdebug.mode
```

qui doit contenir :

```text
develop,debug
```

et :

```text
xdebug.start_with_request
```

qui doit être :

```text
yes
```

Vérifier également :

```text
xdebug.client_host
```

avec :

```text
127.0.0.1
```

et :

```text
xdebug.client_port
```

avec :

```text
9003
```

# 9. Configurer PHP dans PhpStorm

PhpStorm doit connaître l'interpréteur PHP utilisé par le projet.

Dans PhpStorm :

```text
File
  ↓
Settings
  ↓
PHP
```

Dans **CLI Interpreter**, sélectionner l'interpréteur PHP correspondant à WampServer.

Par exemple :

```text
C:\wamp64\bin\php\php8.3.28\php.exe
```

Il est important que cette version corresponde à celle réellement utilisée par l'environnement du projet.

# 10. Vérifier Xdebug dans PhpStorm

Dans :

```text
File
  ↓
Settings
  ↓
PHP
```

PhpStorm doit détecter Xdebug pour l'interpréteur PHP sélectionné.

La configuration doit notamment utiliser :

```text
Xdebug
```

avec le port :

```text
9003
```

Dans :

```text
Settings
→ PHP
→ Debug
```

vérifier :

```text
Xdebug port : 9003
```

Le port doit correspondre à :

```ini
xdebug.client_port=9003
```

dans `php.ini`.

# 11. Configurer le serveur Web dans PhpStorm

PhpStorm doit également savoir à quel serveur Web correspond le projet.

Cette configuration est particulièrement utile lorsque le projet est accessible par un VirtualHost.

Aller dans :

```text
File
  ↓
Settings
  ↓
PHP
  ↓
Servers
```

Ajouter un serveur.

Par exemple :

```text
Name : membre.local
Host : membre.local
Port : 80
Debugger : Xdebug
```

Le nom doit correspondre au serveur utilisé pour accéder au projet.

Par exemple, si le navigateur utilise :

```text
http://membre.local/
```

on peut déclarer :

```text
Host : membre.local
```

# 12. Path Mapping

Dans la configuration du serveur PhpStorm, une option appelée :

```text
Use path mappings
```

peut être proposée.

Le Path Mapping permet d'indiquer à PhpStorm qu'un fichier présent à un emplacement sur le serveur correspond à un fichier situé à un autre emplacement sur la machine de développement.

Cette configuration est surtout nécessaire lorsque les fichiers exécutés par PHP et les fichiers ouverts dans PhpStorm ne sont pas au même emplacement logique.

Dans notre environnement, **si le débogage fonctionne sans Path Mapping, il n'est pas nécessaire de l'activer**.

Il ne faut donc pas ajouter un Path Mapping simplement parce que l'option existe.

Exemple :

```text
Projet local :
D:\Projets\membre

Projet exécuté :
D:\Projets\membre
```

Dans ce cas, aucun mapping particulier n'est généralement nécessaire.

# 13. Activer l'écoute des connexions Xdebug

La configuration de Xdebug ne suffit pas.

PhpStorm doit également être prêt à recevoir les connexions.

Dans PhpStorm :

```text
Run
  ↓
Start Listening for PHP Debug Connections
```

Cette fonction active l'écoute des connexions provenant de Xdebug.

L'état de l'écoute est visible dans la barre d'outils de PhpStorm.

Il faut retenir :

```text
Xdebug actif
        +
PhpStorm en écoute
        ↓
Session de débogage possible
```

Si PhpStorm n'écoute pas, Xdebug ne peut pas transmettre la session de débogage à l'IDE.

# 14. Poser un point d'arrêt

Un **breakpoint** est un point d'arrêt.

Il indique à PhpStorm :

> Lorsque l'exécution arrive ici, arrête le programme afin que je puisse examiner son état.

Dans PhpStorm, cliquer dans la marge située à gauche d'une ligne de code.

Un point rouge apparaît :

```text
🔴
```

Par exemple :

```php
$user = getUser($id);

$name = $user['name'];   // 🔴 breakpoint

echo $name;
```

# 15. Démarrer une session de débogage

Avec la configuration :

```ini
xdebug.start_with_request=yes
```

le démarrage est volontairement simple.

### Étape 1

Ouvrir le projet dans PhpStorm.

### Étape 2

Placer un breakpoint.

### Étape 3

Dans PhpStorm :

```text
Run
  ↓
Start Listening for PHP Debug Connections
```

L'écoute doit être activée.

### Étape 4

Ouvrir l'application dans le navigateur.

Par exemple :

```text
http://membre.local/
```

### Étape 5

Lorsque l'exécution PHP atteint le breakpoint, PhpStorm prend la main.

La ligne concernée est mise en évidence.

Le développeur peut alors :

- consulter les variables ;
    
- utiliser la console ;
    
- avancer d'une ligne ;
    
- entrer dans une fonction ;
    
- sortir d'une fonction ;
    
- poursuivre l'exécution.
    
# 16. Comprendre le point important : écouter ou ne pas écouter

Il existe une différence importante entre :

```text
Xdebug est installé et configuré
```

et :

```text
PhpStorm écoute les connexions de débogage
```

Xdebug peut être présent et correctement configuré sans que PhpStorm ne soit actuellement utilisé pour déboguer.

L'écoute de PhpStorm constitue donc le moyen pratique de décider si l'IDE doit recevoir les connexions de débogage.

# 17. Arrêter une session de débogage

Une session de débogage ne doit pas rester active lorsque l'on souhaite simplement utiliser l'application normalement.

Dans PhpStorm, arrêter l'écoute :

```text
Run
  ↓
Stop Listening for PHP Debug Connections
```

ou désactiver l'icône correspondante dans la barre d'outils.

Une fois l'écoute désactivée, PhpStorm ne prendra plus en charge les nouvelles connexions de débogage.

# 18. Pourquoi faut-il arrêter l'écoute ?

Cette étape est particulièrement importante dans notre configuration.

Nous avons :

```ini
xdebug.start_with_request=yes
```

Cela signifie que les requêtes PHP sont susceptibles de démarrer une communication avec Xdebug.

Si PhpStorm est en écoute et qu'une requête atteint un breakpoint, l'exécution peut être interrompue dans PhpStorm.

On peut alors avoir l'impression que :

```text
http://127.0.0.1/
```

ou :

```text
http://monprojet.local/
```

ne fonctionne plus.

En réalité, PHP est simplement **arrêté par le débogueur**.

Le navigateur attend donc la poursuite de l'exécution.

# 19. Exemple du problème

Supposons que le développeur ait placé un breakpoint dans :

```php
index.php
```

et qu'il laisse PhpStorm en mode :

```text
Start Listening for PHP Debug Connections
```

Il ouvre ensuite :

```text
http://127.0.0.1/
```

ou une autre application PHP.

La requête arrive dans PHP.

Xdebug détecte le débogage.

PhpStorm reçoit la connexion.

Si le code atteint le breakpoint :

```text
Navigateur
    │
    ▼
   PHP
    │
    ▼
 Xdebug
    │
    ▼
PhpStorm
    │
    │ breakpoint
    ▼
  PAUSE
```

Le navigateur attend.

Pour poursuivre :

```text
F8
```

ou le bouton **Resume Program** dans PhpStorm.

Pour arrêter complètement le débogage :

```text
Run
  ↓
Stop Listening for PHP Debug Connections
```

# 20. Bonne pratique au quotidien

Il est recommandé de fonctionner ainsi :

```text
                 BESOIN DE DÉBOGUER ?
                         │
              ┌──────────┴──────────┐
              │                     │
             NON                   OUI
              │                     │
              ▼                     ▼
      PhpStorm n'écoute       Activer l'écoute
                                    │
                                    ▼
                             Poser breakpoint
                                    │
                                    ▼
                              Ouvrir le site
                                    │
                                    ▼
                               Déboguer
                                    │
                                    ▼
                           Travail terminé ?
                                    │
                                    ▼
                       Désactiver l'écoute
```

Cette habitude évite de lancer accidentellement des sessions de débogage pendant une navigation normale.

# 21. Plusieurs manières d'arrêter le débogage

Il faut distinguer deux actions.

## Poursuivre l'exécution

Si PhpStorm est actuellement arrêté sur un breakpoint mais que l'on souhaite continuer :

```text
Resume Program
```

ou :

```text
F8
```

selon la configuration de PhpStorm.

L'application poursuit son exécution.

## Arrêter l'écoute

Si l'on ne souhaite plus déboguer :

```text
Stop Listening for PHP Debug Connections
```

C'est cette seconde action qu'il faut utiliser lorsque l'on repasse en navigation ou en développement normal.

# 22. Configuration recommandée : mode simple

Pour un poste de développement classique, nous recommandons dans un premier temps :

```ini
xdebug.mode=develop,debug
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

Cette configuration présente l'avantage d'être simple à comprendre.

Le développeur n'a pas besoin de gérer un paramètre supplémentaire dans le navigateur.

Il doit simplement :

```text
1. Activer l'écoute dans PhpStorm
2. Poser un breakpoint
3. Ouvrir la page
4. Déboguer
5. Désactiver l'écoute lorsqu'il a terminé
```

# 23. Alternative : utiliser le mode Trigger

Il est également possible de configurer Xdebug pour qu'une session de débogage ne démarre **que lorsqu'un trigger est présent**.

Cette solution est plus sélective.

La configuration devient :

```ini
xdebug.mode=develop,debug
xdebug.start_with_request=trigger

xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

Avec :

```text
xdebug.start_with_request=trigger
```

Xdebug ne démarre pas automatiquement une session de débogage pour chaque requête.

Il faut explicitement demander le démarrage du débogage.

# 24. Pourquoi utiliser Trigger ?

Le mode Trigger peut être intéressant lorsque le développeur utilise très fréquemment son application sans avoir besoin de la déboguer.

On obtient alors :

```text
Navigation normale
        │
        ▼
Pas de session Xdebug
```

et :

```text
Navigation avec trigger
        │
        ▼
Session Xdebug
        │
        ▼
PhpStorm
```

Cela limite les sessions de débogage aux requêtes pour lesquelles le développeur les demande explicitement.

# 25. Utiliser Trigger avec Chrome

Une manière simple d'envoyer le trigger depuis Chrome consiste à utiliser une extension dédiée à Xdebug.

Par exemple, une extension de navigateur permettant d'activer/désactiver le cookie de session Xdebug peut être utilisée.

Le principe est le suivant :

```text
Chrome
  │
  │ activation du trigger
  ▼
Cookie Xdebug
  │
  ▼
Requête HTTP
  │
  ▼
PHP / Xdebug
  │
  ▼
PhpStorm
```

# 26. Installation de l'extension Chrome

Dans Chrome, ouvrir le **Chrome Web Store** et rechercher une extension de type :

```text
Xdebug helper
```

Choisir une extension compatible avec Xdebug 3.

Après installation, une icône de l'extension apparaît dans la barre d'outils du navigateur.

L'objectif de l'extension est de permettre d'ajouter ou de retirer facilement le trigger Xdebug sans modifier manuellement l'URL.

# 27. Activer le trigger dans Chrome

Une fois l'extension installée :

### Étape 1

Ouvrir le projet dans Chrome.

Par exemple :

```text
http://membre.local/
```

### Étape 2

Cliquer sur l'icône de l'extension Xdebug.

### Étape 3

Activer le mode correspondant au débogage.

L'extension ajoute alors le mécanisme nécessaire à la requête pour que Xdebug reconnaisse le trigger.

### Étape 4

Dans PhpStorm, activer :

```text
Run
  ↓
Start Listening for PHP Debug Connections
```

### Étape 5

Recharger la page.

Si un breakpoint est rencontré, PhpStorm interrompt l'exécution.

# 28. Arrêter le trigger dans Chrome

Une fois le débogage terminé, désactiver le mode de débogage dans l'extension.

Il faut également penser à désactiver l'écoute dans PhpStorm :

```text
Run
  ↓
Stop Listening for PHP Debug Connections
```

La situation normale redevient alors :

```text
Chrome
  │
  ▼
Application PHP
  │
  ▼
Pas de session de débogage
```

# 29. Comparaison des deux modes

|Critère|`yes`|`trigger`|
|---|---|---|
|Configuration|Simple|Plus complexe|
|Débogage|Activé pour les requêtes|Activé uniquement avec le trigger|
|Extension navigateur|Non nécessaire|Recommandée|
|Utilisation quotidienne|Très simple|Plus contrôlée|
|Risque de lancer involontairement Xdebug|Plus important|Faible|
|Recommandation|**Configuration de départ**|Configuration avancée|

Pour notre environnement, la configuration recommandée pour commencer est donc :

```
xdebug.start_with_request=yes
```

```ini
xdebug.start_with_request=yes
```

Le mode `trigger` peut être adopté ultérieurement si le développeur souhaite contrôler plus finement quelles requêtes doivent être déboguées.

# 30. Consulter le journal Xdebug

En cas de problème, Xdebug peut écrire des informations dans un fichier de log.

Dans notre configuration :

```text
C:\wamp64\logs\xdebug.log
```

La configuration correspondante est :

```ini
xdebug.log="c:/wamp64/logs/xdebug.log"
xdebug.log_level=7
```

Le niveau `7` permet d'obtenir des informations détaillées utiles au diagnostic.

# 31. Vérifier que Xdebug tente de contacter PhpStorm

Dans le fichier de log, on peut notamment rechercher des informations indiquant que Xdebug tente d'établir une connexion avec :

```text
127.0.0.1:9003
```

Si Xdebug tente de se connecter mais que PhpStorm ne reçoit rien, vérifier :

```text
1. PhpStorm écoute-t-il ?
2. Le port est-il bien 9003 ?
3. Le pare-feu bloque-t-il la connexion ?
4. Le PHP utilisé est-il bien celui configuré ?
```

# 32. Vérifier le port 9003

Depuis un terminal Windows :

```cmd
netstat -ano | findstr 9003
```

Cette commande permet notamment de vérifier l'utilisation du port `9003`.

Le port doit être cohérent entre :

```text
php.ini
```

et :

```text
PhpStorm
```

# 33. Problème : PhpStorm ne s'arrête pas sur le breakpoint

Vérifier dans l'ordre :

### Xdebug est-il chargé ?

Consulter `phpinfo()`.

### Le mode debug est-il activé ?

Vérifier :

```ini
xdebug.mode=develop,debug
```

### Le démarrage est-il configuré ?

Pour la configuration simple :

```ini
xdebug.start_with_request=yes
```

### PhpStorm écoute-t-il ?

Vérifier :

```text
Start Listening for PHP Debug Connections
```

### Le port est-il correct ?

```text
9003
```

### Le breakpoint est-il placé dans du code réellement exécuté ?

Un breakpoint placé dans une fonction qui n'est jamais appelée ne sera évidemment jamais atteint.

# 34. Problème : le navigateur semble bloqué

Si le navigateur attend indéfiniment et que PhpStorm affiche une session de débogage, vérifier si PHP est simplement arrêté sur un breakpoint.

Dans ce cas :

```text
Resume Program
```

permet de poursuivre l'exécution.

Si aucun débogage n'est souhaité :

```text
Stop Listening for PHP Debug Connections
```

permet de désactiver l'écoute.

Ce point est particulièrement important lorsque :

```ini
xdebug.start_with_request=yes
```

est utilisé.
# 35. Problème : aucune connexion Xdebug

Vérifier :

```text
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

puis :

```text
PhpStorm → Settings → PHP → Debug
```

et vérifier que le port est également :

```text
9003
```

Consulter ensuite :

```text
C:\wamp64\logs\xdebug.log
```

# 36. Configuration finale recommandée

Pour commencer, utiliser :

```ini
[xdebug]

zend_extension="c:/wamp64/bin/php/php8.3.28/zend_ext/php_xdebug-3.4.7-8.3-ts-vs16-x86_64.dll"

xdebug.mode=develop,debug
xdebug.start_with_request=yes

xdebug.client_host=127.0.0.1
xdebug.client_port=9003

xdebug.output_dir="c:/wamp64/tmp"

xdebug.log="c:/wamp64/logs/xdebug.log"
xdebug.log_level=7

xdebug.show_local_vars=0
xdebug.profiler_output_name=trace.%H.%t.%p.cgrind
xdebug.use_compression=false
```

Après modification :

```text
1. Enregistrer php.ini
2. Redémarrer Apache
3. Vérifier Xdebug avec phpinfo()
4. Vérifier PhpStorm
5. Activer l'écoute
6. Poser un breakpoint
7. Ouvrir le projet
8. Vérifier que PhpStorm s'arrête
9. Désactiver l'écoute après le débogage
```

---

# 37. Procédure quotidienne recommandée

Pour un développement normal :

```text
             TRAVAIL NORMAL
                   │
                   ▼
        PhpStorm n'écoute pas
                   │
                   ▼
             Naviguer
                   │
                   ▼
          Besoin de déboguer ?
                   │
                  OUI
                   │
                   ▼
       Start Listening for
       PHP Debug Connections
                   │
                   ▼
          Poser breakpoint
                   │
                   ▼
            Charger la page
                   │
                   ▼
             Déboguer
                   │
                   ▼
              Terminé ?
                   │
                  OUI
                   │
                   ▼
        Stop Listening for
        PHP Debug Connections
                   │
                   ▼
             Travail normal
```

Cette procédure permet de conserver une configuration Xdebug simple tout en évitant qu'une navigation normale soit interrompue par une session de débogage.

---

# 38. À retenir

Les éléments essentiels sont :

```text
WampServer
    │
    ├── Apache
    │
    └── PHP
          │
          └── Xdebug
                  │
                  ▼
               PhpStorm
```

Pour une première configuration, utiliser :

```ini
xdebug.mode=develop,debug
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

Puis :

```text
Besoin de déboguer
        ↓
Start Listening
        ↓
Breakpoint
        ↓
Navigation
        ↓
Débogage
        ↓
Stop Listening
```

Le mode :

```ini
xdebug.start_with_request=trigger
```

constitue une alternative plus sélective lorsque l'on souhaite décider précisément quelles requêtes doivent démarrer une session Xdebug.

Dans ce cas, une extension Chrome compatible avec Xdebug 3 peut être utilisée pour activer et désactiver facilement le trigger.