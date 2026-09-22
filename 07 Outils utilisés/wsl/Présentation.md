
WSL2 permet d’exécuter un vrai noyau Linux dans Windows grâce à une **machine virtuelle légère**.
- Intégration native Windows
- Performances proches du natif Linux
- Compatible Docker Desktop
- Accès fichiers Windows ↔ Linux

Les fichiers Linux sont stockés dans un disque virtuel :

```
C:\Users<user>\AppData\Local\Packages\
```

ou (Docker Desktop / distros modernes) :

```
C:\Users<user>\AppData\Local\Docker\wsl\
```

Fichier principal :
```
ext4.vhdx
```

# Principales commandes

- Lister les distributions installées : **wsl --list --verbose**
- Voir uniquement les distros installées : **wsl -l**
- Voir la distribution par défaut : **wsl --status**
- Démarrer / utiliser  WSL : wsl
- Lancer une distribution précise : **wsl -d Ubuntu**
- Exécuter une commande Linux depuis Windows : **wsl ls -la**
- Supprimer une distribution (IMPORTANT) : ** wsl --unregister Ubuntu**

👉 Cela :
- supprime complètement la distro
- supprime le système de fichiers Linux
- supprime les données (ext4.vhdx associé)

- Installer une distribution : **wsl --install -d Ubuntu**
- Arrêter toutes les distributions :  **wsl --shutdown**

- Redémarrer une distribution : **wsl -d Ubuntu**


#  Stockage (important)

WSL2 utilise un disque virtuel : ext4.vhdx

Emplacement typique :  C:\Users\<user>\AppData\Local\Packages\ouC:\Users\<user>\AppData\Local\Docker\wsl\

#  Commandes utiles de diagnostic

- Version WSL : **wsl --version**
- État global : **wsl --status**
- Liste des distributions disponibles en installation : **wsl --list --online**

# Points importants

- `wsl --unregister` = suppression totale (équivalent format disque)
- `wsl --shutdown` = arrêt propre de toutes les distros
- Les fichiers WSL sont stockés dans un **VHDX** (disque virtuel)
- La taille du VHDX ne diminue pas automatiquement

#  Résumé rapide

|Action|Commande|
|---|---|
|Lister distribution|`wsl -l -v`|
|Lancer WSL|`wsl`|
|Lancer Ubuntu|`wsl -d Ubuntu`|
|Arrêter WSL|`wsl --shutdown`|
|Supprimer Ubuntu|`wsl --unregister Ubuntu`|
|Installer Ubuntu|`wsl --install -d Ubuntu`|
