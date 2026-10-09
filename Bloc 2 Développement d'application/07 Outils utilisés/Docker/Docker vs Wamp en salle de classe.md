
#  1. Wamp (ou XAMPP) : ce qu’il fait mieux

## ✔ Très simple à utiliser

- installer → ça marche
- PHP + Apache + MySQL déjà prêts
- pas de notion de conteneur
- pas de complexité réseau

## ✔ Debug et IDE faciles

- PhpStorm trouve PHP directement
- Xdebug souvent plus simple à configurer (quand ça marche)

## ✔ Aucun “bruit technique”

Les étudiants se concentrent sur  PHP / SQL / MVC

---

## ❌ Limites de Wamp

Il faut savoir reproduire le même environnement chez zoi

---

# 🟩 2. Docker : ce qu’il fait mieux

## ✔ Environnement identique partout

- même PHP
- même MySQL
- même config

---

## ✔ Projet portable

- zip du projet = environnement complet
- fonctionne sur n’importe quel PC

---

## ✔ Séparation propre

- Apache ≠ MySQL ≠ outils CLI
- architecture professionnelle

---

## ✔ Très proche du monde réel

- dev moderne (entreprise)
- microservices, CI/CD, etc.

---

## ✔ Pas d’installation locale

- pas besoin de Wamp
- pas de conflit de versions PHP

---

## Des inconvénients majeurs

### 1. Alourdissement considérable du profil utilisateur

Le fichier C:\Users\<user>\AppData\Local\Docker\wsl\data\ext4.vhdx est **dans le profil utilisateur**, donc :

- il est **roaming/local profile bloating**
- il suit potentiellement l’utilisateur (selon gestion AD/roaming)
- il peut **gonfler à 10–50 Go facilement**
- il n’est pas nettoyé automatiquement

### 2. Complexité cognitive inutile au début

Il faut comprendre :

- containers
- volumes
- ports
- build
- réseau interne

---

### 3. Debug plus compliqué

- Xdebug
- mapping de volumes
- host.docker.internal
- ports

---

### 4. Erreurs “invisibles”

- conteneur lancé mais PHP cassé
- port déjà utilisé
- mauvais mapping

Cela peut entrainer une frustration élevée

---


### 5. Installation et maintenance en salle de classe

- Docker Desktop parfois instable
- droits admin
- antivirus
- virtualisation BIOS

---

### 6. Fichier de configuration à adapter

Il faut remplacer 'localhost' par 'mysql' dans  config/database.php ainsi que la valeur de password

### 8. Alourdissement du profil Windows

	

#  4. Conclusion 

### Wamp est :

✔ plus rapide à démarrer  
✔ plus simple pédagogiquement  
✔ plus stable pour débutants

---

### Docker est :

✔ plus professionnel  
✔ plus propre en architecture  
✔ meilleur pour travail en équipe / projets avancés

---

