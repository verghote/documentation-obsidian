
Ce document regroupe les principales configurations utiles de **PhpStorm** pour un environnement de développement Web (PHP / MySQL / JavaScript), ainsi que quelques réglages complémentaires importants.

## 1. Licences JetBrains

### Licence réseau (école / entreprise)
Accès via un serveur de licence JetBrains :

https://saintremi.fls.jetbrains.com

### Licence GitHub Education
Permet d’obtenir une licence gratuite pour les étudiants :

https://github.com/settings/education/benefits?locale=en-US

## 2. Configuration de la base de données MySQL

### Ajout d’une source de données
1. Ouvrir l’icône **Database** dans la barre latérale gauche
2. Cliquer sur **+**
3. Sélectionner **Data Source → MySQL (MySQL 8)**
### Paramètres de connexion
- **Host** : `localhost`
- **User** : `root`
- **Password** : laisser vide (ou selon configuration locale)

## 3. Raccourcis utiles

- **Mettre en minuscule / majuscule** : `Ctrl + Shift + U`
- **Reformater le code** : `Ctrl + Shift + L`

## 4. Zoom avec la molette de la souris

Activer le zoom dans l’éditeur :

**File → Settings → Editor → General**  
Cocher : *Change font size with Ctrl + Mouse Wheel*

## 5. Configuration du dialecte SQL

**File → Settings → Languages & Frameworks → SQL Dialects**

- **Global SQL Dialect** : MySQL
- **Project SQL Dialect** : MySQL

## 6. Exécution des scripts SQL

### Attacher une source de données
- Ouvrir un fichier `.sql`
- Si la barre d’exécution n’apparaît pas :
  - clic droit sur le fichier → **Attach Data Source**
### Paramétrage de l’exécution

**File → Settings → Tools → Database → Query Execution**

- When caret inside statement execute : **Whole script**
- When caret outside statement execute : **Whole script**
- For selection execute : **Exactly as separate statements**

## 7. Configuration de l’interpréteur PHP (CLI)

**File → Settings → PHP → CLI Interpreter**

- Ajouter un interpréteur local
- Chemin exemple :


## 8. Qualité du code JavaScript (JSHint)  

### Activation  
  
**File → Settings → Languages & Frameworks → Code Quality Tools → JSHint**  
  
- ✔ Enable  
- ✔ Use config files  
- ✔ Custom configuration file  
### Fichier de configuration

## 9. Désactivation des “inlay hints”  
  
Permet de supprimer les indications automatiques dans le code :  
  
**File → Settings → Editor → Inlay Hints**  
  
- Décocher toutes les cases  
  
## 10. Dictionnaire français  
  
Pour améliorer la correction orthographique :  
  
**File → Settings → Editor → Natural Languages → Grammar and Style**  
  
- Ajouter le dictionnaire français  
  
 
## 11. Inspections (analyse du code)  
  
**File → Settings → Editor → Inspections**  
  
- Permet d’activer/désactiver les analyses automatiques  
- Détection des erreurs, warnings et mauvaises pratiques  
  
## 12. Désactivation des conseils au démarrage  
  
Pour éviter les popups d’aide :  
  
**File → Settings → Appearance & Behavior → System Settings**  
  
- Désactiver les tips / suggestions au lancement  
  
## 13.  désactiver l'inspection qui signale les exceptions non gérées.

**Settings → Editor → Inspections → PHP → Error handling → Unhandled exception**

## 14. Bonus — bonnes pratiques utiles  
  
### 🔧 Auto-save et sécurité  
- Vérifier que **Autosave** est activé  
- Activer **Local History** (très utile pour revenir en arrière)  
  
### 🎨 Lisibilité du code  
- Choisir un thème adapté (Dark / Light)  
- Ajuster la taille de police : Editor → Font  
  
### ⚡ Productivité  
- Utiliser **Live Templates**  
- Configurer **File Watchers** (SASS, JS, etc.)  
  
