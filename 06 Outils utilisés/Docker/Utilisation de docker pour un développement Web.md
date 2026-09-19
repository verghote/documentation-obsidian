# 🧱 Création d'un fichier `docker-compose.yml` à la racine du projet

Ce fichier décrit **l’ensemble des services nécessaires au projet web** (serveur web, base de données, outils).

---

## ⚙️ Contenu du fichier

```
services:  
  
apache:  
build: .  
container_name: apache  
volumes:  
- ./:/var/www/html  
ports:  
- "8080:80"  
depends_on:  
- mysql  
  
  
mysql:  
image: mysql:8.4  
container_name: mysql  
environment:  
MYSQL_ROOT_PASSWORD: root  
ports:  
- "3306:3306"  
volumes:  
# Persistance des données (BDD conservée même après arrêt)  
- ./data/mysql:/var/lib/mysql  
  
  
phpmyadmin:  
image: phpmyadmin/phpmyadmin  
container_name: phpmyadmin  
ports:  
- "8081:80"  
environment:  
PMA_HOST: mysql  
  
  
php-cli:  
image: php:8.3-cli  
container_name: php_cli  
working_dir: /var/www/html  
volumes:  
- ./:/var/www/html  
  
  
volumes:  
mysql_data:
```

---

#  Explication du contenu

---

##  Service `apache`

- Contient **PHP + Apache**
- Sert à exécuter le site web
- Accessible via :

```
http://localhost:8080
```

- `volumes` :
    - synchronise le code local avec le conteneur
- `build: .` :
    - utilise le Dockerfile du projet

---

## Service `mysql`

- Base de données MySQL 8.4
- Stocke les données du projet

### Configuration :

- utilisateur root
- mot de passe : `root`

### Port :

```
3306
```

### Persistance :

```
- ./data/mysql:/var/lib/mysql
```

👉 les données restent même si on supprime le conteneur

---

## Service `phpmyadmin`

- Interface graphique pour MySQL
- Permet de gérer la base sans ligne de commande

Accès :

```
http://localhost:8081
```

---

## Service `php-cli`

- Permet d’exécuter PHP en ligne de commande
- Utilisé pour :
    - scripts PHP
    - migrations
    - Composer

Exemple :

```
docker compose run php-cli php script.php
```

---

# 🚀 Création des conteneurs

```
docker compose up -d --build
```

---

## 📌 Explication

Cette commande :

- construit les images (`--build`)
- crée les conteneurs
- démarre les services en arrière-plan (`-d`)

---

# ▶️ Commandes utiles

---

## Démarrer les services

```
docker compose up -d
```

---

## Arrêter les services

```
docker compose down
```

---

## Rebuild complet

```
docker compose up -d --build
```

---

## Voir les conteneurs actifs

```
docker ps
```

---

## Exécuter un script PHP (CLI)

```
docker compose run php-cli php src/script.php
```

---

## Installer Composer (si ajouté)

```
docker compose run composer install
```

---

# Adaptation du fichier de configuration config/database.php

```
<?php  
return [  
    'host' => 'mysql',  
    'database' => 'consultation',  
    'user' => 'root',  
    'password' => 'root',  
    'port' => 3306  
];
```

#  Conclusion

Docker permet de :

- standardiser l’environnement
- éviter les installations locales (Wamp, XAMPP)
- garantir que le projet fonctionne partout
- séparer clairement les rôles des services

