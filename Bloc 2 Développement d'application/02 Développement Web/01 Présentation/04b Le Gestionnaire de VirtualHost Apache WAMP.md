### 1. Structure du projet

```text
Apache/
│
├── apache.py
├── apache_utils.py
├── backup_utils.py
├── vhost_utils.py
├── hosts_utils.py
├── config.py
│
└── apache.bat
```

Le programme est conçu pour gérer les VirtualHosts Apache d'une installation WAMP sous Windows.

Il permet notamment de :

- créer un VirtualHost ;
- supprimer un VirtualHost ;
- lister les VirtualHosts existants ;
- démarrer Apache ;
- arrêter Apache ;
- redémarrer Apache ;
- vérifier l'état d'Apache ;
- modifier automatiquement le fichier `hosts` de Windows ;
- sauvegarder les configurations avant la création d'un VirtualHost.

# 2. `apache.py`

### Rôle

`apache.py` est **le programme principal**.

C'est le fichier que l'utilisateur lance pour accéder au menu de gestion.

Il contient l'interface utilisateur et orchestre les différentes opérations.

### Fonctions principales

Le menu principal permet :

```text
1. Créer un VirtualHost
2. Supprimer un VirtualHost
3. Lister les VirtualHosts
4. Gestion Apache
5. Quitter
```

Lors de la création d'un VirtualHost, `apache.py` :

1. demande le nom du domaine ;
2. vérifie que le projet existe ;
3. vérifie la présence du dossier `public` ;
4. sauvegarde les fichiers de configuration ;
5. crée le VirtualHost Apache ;
6. ajoute le domaine dans `hosts` ;
7. redémarre Apache.

### Dépendances

`apache.py` utilise :

```text
apache_utils.py
backup_utils.py
hosts_utils.py
vhost_utils.py
config.py
```

Il constitue donc **le point central du programme**.

# 3. `apache_utils.py`

### Rôle

`apache_utils.py` contient tout ce qui concerne **le service Apache Windows**.

Il ne gère pas les VirtualHosts directement.

### Fonctions principales

Il fournit notamment :

```text
require_admin()
start_apache()
stop_apache()
restart_apache()
apache_status()
```

### Administrateur Windows

Le module vérifie également si le programme possède les droits administrateur grâce à :

```python
ctypes.windll.shell32.IsUserAnAdmin()
```

Cette vérification est nécessaire car le programme doit notamment pouvoir :

- contrôler le service Apache ;
- modifier le fichier `hosts` de Windows ;
- modifier la configuration Apache.

### Dépendance

`apache_utils.py` utilise :

```text
config.py
```

pour connaître le nom du service Apache :

```text
wampapache64
```


# 4. `backup_utils.py`

### Rôle

`backup_utils.py` est responsable de la **sauvegarde des configurations avant toute création de VirtualHost**.

Lorsqu'un VirtualHost est créé, le programme sauvegarde :

```text
hosts
httpd-vhosts.conf
```

### Organisation des sauvegardes

Les sauvegardes sont stockées dans :

```text
backup_virtual_host/
```

Chaque opération crée son propre répertoire avec la date et l'heure.

Exemple :

```text
backup_virtual_host/
└── 2026-09-11_05-51-40/
    ├── hosts
    └── httpd-vhosts.conf
```

Cela permet de conserver l'état des fichiers **avant la modification**.

### Dépendance

`backup_utils.py` utilise :

```text
config.py
```

pour connaître les emplacements des fichiers à sauvegarder.

# 5. `vhost_utils.py`

### Rôle

`vhost_utils.py` est responsable de la **gestion de la configuration des VirtualHosts Apache**.

Il travaille directement avec :

```text
httpd-vhosts.conf
```

### Fonctions principales

Il fournit notamment :

```text
get_vhosts()
vhost_exists()
add_vhost()
remove_vhost()
```

### Création

Lorsqu'un VirtualHost est créé, le module génère un bloc Apache similaire à :

```apache
<VirtualHost *:80>
    ServerName test
    DocumentRoot "J:/VirtualHostSlam/test/public"

    <Directory "J:/VirtualHostSlam/test/public/">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Suppression

La suppression recherche le `ServerName` correspondant et supprime **uniquement le bloc VirtualHost concerné**.

### Dépendance

`vhost_utils.py` utilise :

```text
config.py
```

notamment pour connaître :

```text
APACHE_VHOSTS
SYSTEM_DOMAINS
```

# 6. `hosts_utils.py`

### Rôle

`hosts_utils.py` gère le fichier `hosts` de Windows.

Il permet d'associer un domaine local à la machine.

Lors de la création d'un VirtualHost, deux entrées sont utilisées :

```text
127.0.0.1    test
::1          test
```

La première correspond à IPv4 et la seconde à IPv6.

### Fonctions principales

Le module permet notamment de :

```text
host_exists()
host_mapping_exists()
add_host()
remove_host()
```

### Ajout

Lorsqu'un VirtualHost `test` est créé, le fichier `hosts` doit contenir :

```text
127.0.0.1    test
::1          test
```

Si une entrée existe déjà, elle n'est pas ajoutée une seconde fois.

### Suppression

Lors de la suppression du VirtualHost `test`, les deux associations sont supprimées.

La comparaison est exacte.

Ainsi :

```text
test
test-dev
test-production
```

sont considérés comme trois domaines différents.

### Dépendance

`hosts_utils.py` utilise :

```text
config.py
```

pour connaître :

```text
HOSTS_FILE
LOCAL_IP
```

# 7. `config.py`

### Rôle

`config.py` contient **toute la configuration générale du programme**.

L'objectif est d'éviter de disperser les chemins et paramètres dans les différents modules.

Il contient notamment :

```text
APACHE_VHOSTS
HOSTS_FILE
APACHE_SERVICE
LOCAL_IP
SYSTEM_DOMAINS
```

### Configuration actuelle

Le fichier définit notamment :

```text
httpd-vhosts.conf
C:\Windows\System32\drivers\etc\hosts
wampapache64
127.0.0.1
localhost
```

C'est donc le fichier à modifier si l'installation WAMP change de chemin ou si le nom du service Apache change.

# 8. `apache.bat`

### Rôle

`apache.bat` est le **point de lancement Windows** du programme.

Son rôle est très simple :

1. demander les droits administrateur ;
2. lancer `apache.py`.

L'utilisateur n'a donc pas besoin d'ouvrir manuellement un terminal en administrateur.

Le fichier peut être :

```bat
@echo off

powershell -Command "Start-Process py -ArgumentList 'apache.py' -Verb RunAs"

exit
```

Lorsque l'utilisateur double-clique sur :

```text
apache.bat
```

Windows demande l'autorisation administrateur puis lance :

```text
apache.py
```

dans une nouvelle console avec les privilèges nécessaires.

# 9. Imbrication des fichiers

L'architecture générale est la suivante :

```text
                         apache.bat
                              │
                              ▼
                         apache.py
                              │
              ┌───────────────┼────────────────┐
              │               │                │
              ▼               ▼                ▼
       apache_utils.py   backup_utils.py   vhost_utils.py
              │               │                │
              │               │                │
              ▼               ▼                ▼
                         config.py
                              ▲
                              │
                       hosts_utils.py
```

Plus simplement :

```text
apache.bat
    │
    ▼
apache.py
    │
    ├──► apache_utils.py
    │         │
    │         └──► config.py
    │
    ├──► backup_utils.py
    │         │
    │         └──► config.py
    │
    ├──► vhost_utils.py
    │         │
    │         └──► config.py
    │
    └──► hosts_utils.py
              │
              └──► config.py
```

## 10. Principe général

Chaque fichier possède donc une responsabilité précise :

|Fichier|Responsabilité|
|---|---|
|`apache.bat`|Lancement du programme en administrateur|
|`apache.py`|Interface et orchestration générale|
|`apache_utils.py`|Gestion du service Apache|
|`backup_utils.py`|Sauvegarde des configurations|
|`vhost_utils.py`|Gestion des VirtualHosts Apache|
|`hosts_utils.py`|Gestion du fichier `hosts` Windows|
|`config.py`|Configuration centralisée|

L'objectif de cette organisation est de **séparer les responsabilités** : `apache.py` orchestre, tandis que les différents modules spécialisés effectuent réellement les opérations.

## 11. Exemple du processus de création

Lorsqu'on demande :

```text
Créer un VirtualHost
```

pour le domaine :

```text
test
```

le processus est :

```text
apache.py
    │
    ├──► backup_utils.py
    │       └── sauvegarde hosts
    │       └── sauvegarde httpd-vhosts.conf
    │
    ├──► vhost_utils.py
    │       └── ajout du VirtualHost
    │
    ├──► hosts_utils.py
    │       ├── ajout de 127.0.0.1 test
    │       └── ajout de ::1 test
    │
    └──► apache_utils.py
            └── redémarrage d'Apache
```

Le résultat attendu est donc :

```text
httpd-vhosts.conf
        │
        └── VirtualHost test

hosts
        │
        ├── 127.0.0.1 test
        └── ::1 test

Apache
        │
        └── redémarré
```

Et le site devient accessible avec :

```text
http://test
```

Cette architecture est maintenant suffisamment modulaire pour que les futures évolutions puissent être ajoutées sans transformer `apache.py` en un fichier contenant toute la logique.