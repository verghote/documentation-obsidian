
# fichier docker-compose.yml

```
 php-cli:  
  image: php:8.3-cli  
  working_dir: /var/www/html  
  volumes:  
    - ./:/var/www/html
```

---

# 🚀 4. COMMENT ON L’UTILISE

## 👉 Installer les dépendances

```
docker compose run composer install
```

---

## 👉 Ajouter une librairie

```
docker compose run composer require monolog/monolog
```

---

## 👉 Mettre à jour

```
docker compose run composer update
```