## 1. Concepts essentiels

### Image

Une **image** est un modèle prêt à l'emploi contenant une application et ses dépendances.

Exemples :

- `nginx`
- `postgres`
- `ghcr.io/open-webui/open-webui:main`

Lister les images :

```
docker images
```

---

### Conteneur

Un **conteneur** est une instance en cours d'exécution d'une image.

Lister les conteneurs actifs :

```
docker ps
```

Lister tous les conteneurs :

```
docker ps -a
```

---

### Volume

Un **volume** permet de conserver les données même si le conteneur est supprimé.

Lister les volumes :

```
docker volume ls
```

---

## 2. Cycle de vie d'un conteneur

### Télécharger et lancer

```
docker run nginx
```

### Lancer en arrière-plan

```
docker run -d nginx
```

### Nommer un conteneur

```
docker run -d --name mon-nginx nginx
```

### Arrêter

```
docker stop mon-nginx
```

### Démarrer

```
docker start mon-nginx
```

### Redémarrer

```
docker restart mon-nginx
```

### Supprimer

```
docker rm mon-nginx
```

---

## 3. Gestion des ports

Exposer un port :

```
docker run -d -p 8080:80 nginx
```

Signification :

```
8080 = port du PC80   = port du conteneur
```

Accès :

```
http://localhost:8080
```

---

## 4. Volumes (persistance)

Créer un volume :

```
docker volume create mes-donnees
```

Utiliser un volume :

```
docker run -d \-v mes-donnees:/data \mon-image
```

Supprimer :

```
docker volume rm mes-donnees
```

---

## 5. Open WebUI

Installation :

```
docker run -d \-p 3000:8080 \-v open-webui:/app/backend/data \--name open-webui \ghcr.io/open-webui/open-webui:main
```

Accès :

```
http://localhost:3000
```

---

## 6. Logs

Voir les journaux :

```
docker logs open-webui
```

Suivre les logs en temps réel :

```
docker logs -f open-webui
```

---

## 7. Entrer dans un conteneur

Ouvrir un terminal :

```
docker exec -it open-webui bash
```

ou

```
docker exec -it open-webui sh
```

---

## 8. Nettoyage

### Supprimer un conteneur

```
docker rm nom-conteneur
```

### Supprimer une image

```
docker rmi nom-image
```

### Supprimer un volume

```
docker volume rm nom-volume
```

### Tout nettoyer

```
docker system prune
```

Nettoyage complet :

```
docker system prune -a
```

⚠️ Supprime toutes les images inutilisées.

---

## 9. Diagnostic rapide

Version Docker :

```
docker version
```

Informations système :

```
docker info
```

État des conteneurs :

```
docker ps -a
```

Consommation :

```
docker stats
```

---

## 10. Les 5 commandes à retenir

```
docker psdocker imagesdocker rundocker stopdocker logs
```

## Exemple concret : Open WebUI

Créer :

```
docker run -d -p 3000:8080 -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
```

Arrêter :

```
docker stop open-webui
```

Redémarrer :

```
docker start open-webui
```

Voir les logs :

```
docker logs open-webui
```

Supprimer complètement :

```
docker stop open-webuidocker rm open-webuidocker volume rm open-webuidocker rmi ghcr.io/open-webui/open-webui:main
```

### Règle simple à retenir

- **Image** = le programme.
- **Conteneur** = le programme qui tourne.
- **Volume** = les données conservées.
- **Port** = la porte d'entrée depuis votre PC.
- **Docker Desktop** = le moteur qui fait fonctionner tout cela sous Windows.

