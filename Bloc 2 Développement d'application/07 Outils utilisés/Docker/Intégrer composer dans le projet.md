
# fichier docker-compose.yml

```
  composer:  
	image: composer:2  
	container_name: consultation_composer  
	volumes:  
		- ./:/app  
	working_dir: /app
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