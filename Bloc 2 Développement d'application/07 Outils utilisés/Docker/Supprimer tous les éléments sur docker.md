Pour être sûr à 100% qu’il ne reste **aucune trace Docker (conteneurs, images, volumes, cache)**, il faut faire un nettoyage complet et vérifier.

---

# 🧹 1. Arrêter tous les conteneurs

```
docker stop $(docker ps -aq)
```

---

# 🗑️ 2. Supprimer tous les conteneurs

```
docker rm $(docker ps -aq)
```

---

# 🖼️ 3. Supprimer toutes les images

```
docker rmi -f $(docker images -aq)
```

---

# 💾 4. Supprimer tous les volumes (TRÈS IMPORTANT pour MySQL)

```
docker volume rm $(docker volume ls -q)
```

---

# 🌐 5. Supprimer les réseaux inutilisés

```
docker network prune -f
```

---

# 💥 6. Nettoyage TOTAL (commande unique)

👉 c’est la méthode la plus simple :

```
docker system prune -a --volumes -f
```

---

# 🧠 7. Vérification (très important)

## Conteneurs :

```
docker ps -a
```

👉 doit être vide

---

## Images :

```
docker images
```

👉 doit être vide

---

## Volumes :

```
docker volume ls
```

👉 doit être vide

---

## Réseaux :

```
docker network ls
```

👉 seulement :

- bridge
- host
- none

---

# 🪟 8. Vérification Windows (Docker Desktop)

Tu peux aussi vérifier :

```
Docker Desktop → Troubleshoot → Clean / Purge data
```

👉 ou “Reset to factory defaults”

---

# ⚠️ 9. Attention importante

❌ cette commande supprime TOUT :

```
docker system prune -a --volumes
```

👉 tu perds :

- images
- bases MySQL Docker
- caches Composer Docker

---

# 🧠 10. Conclusion simple

✔ Pour être sûr à 100% :

1. `docker system prune -a --volumes -f`
2. vérifier `docker ps -a`
3. vérifier `docker images`
4. vérifier `docker volume ls`

---

# 🚀 11. Résumé mental

👉 Docker “propre” =

```
aucun conteneuraucune imageaucun volume
```