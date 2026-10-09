## 1. Exécuter directement un script `.py`

### Prérequis

Pour exécuter un script Python directement depuis Windows, il faut disposer de :

- **Python** installé ;
- **Python Launcher** installé ;
- les bibliothèques Python utilisées par le script.

Le **Python Launcher** permet notamment d'utiliser la commande :

py

et facilite l'exécution des scripts Python sous Windows.


## 2. Vérifier l'installation

Ouvrir une fenêtre **Invite de commandes** (`cmd`) et exécuter :

```batch
py --version
py -m pip --version
```


Exemple :  Python 3.14.7 - pip 26.0.1 from C:\Users\guyve\AppData\Roaming\Python\Python314\site-packages\pip (python 3.14)

## 3. Installer les bibliothèques utilisées

Un script Python peut utiliser des bibliothèques qui ne font pas partie de Python.

Par exemple, si le script contient :

from colorama import init, Fore

la bibliothèque `colorama` doit être installée.

Installation :

```
py -m pip install colorama
```


Afficher toutes les bibliothèques installées

```
py -m pip list
```

### Plusieurs bibliothèques

Si le programme utilise par exemple :

import requests
import mysql.connector
from colorama import Fore

on peut installer les bibliothèques nécessaires avec :

```
py -m pip install requests mysql-connector-python colorama
```

## 4.. Exécuter le script

Depuis une invite de commandes : py mon_script.py

Si l'association des fichiers `.py` est correctement configurée avec Python, un **double-clic sur le fichier `.py` depuis l'Explorateur Windows** peut également lancer le programme.

### Attention aux programmes en console

Si le script est lancé par double-clic et qu'il se termine immédiatement, la fenêtre de commande peut disparaître avant que l'on puisse lire les messages.

Pour un programme destiné à être utilisé par double-clic, on peut par exemple terminer le programme par :

input("Appuyez sur Entrée pour quitter...")

# 5. Générer un exécutable Windows `.exe`

Une autre solution consiste à transformer le programme Python en **exécutable Windows autonome**.

Une solution couramment utilisée est **PyInstaller**.

## Installation

Sur le poste de développement :

py -m pip install pyinstaller

Puis, depuis le répertoire contenant le script :

pyinstaller --onefile mon_script.py

Par exemple :

pyinstaller --onefile sauvegarde_mysql.py

PyInstaller génère notamment :

 ```
 mon_projet\
│
├── sauvegarde_mysql.py
├── sauvegarde_mysql.spec
│
├── build\
│
└── dist\
    └── sauvegarde_mysql.exe
 
 ```


Le fichier à distribuer est généralement : dist\sauvegarde_mysql.exe

# 6. Avantages de l'exécutable `.exe`

### Pas besoin d'installer Python

L'utilisateur final n'a normalement pas besoin d'installer :

- Python ;
- Python Launcher ;
- `pip`.

Il lance simplement  sauvegarde_mysql.exe par double-clic.
### Les bibliothèques Python sont intégrées

Les dépendances utilisées par le programme sont généralement intégrées dans l'application générée.

L'utilisateur n'a donc pas besoin de faire : py -m pip install colorama

### Installation simplifiée

Pour un petit outil destiné à quelques utilisateurs, on peut simplement fournir :

sauvegarde_mysql.exe

et éventuellement les fichiers nécessaires à son fonctionnement.

# 7. Inconvénients de l'exécutable `.exe`

## Taille du fichier

C'est un point important.

Même pour un script Python de quelques dizaines de lignes, l'exécutable peut être **beaucoup plus volumineux que le script `.py`**, car il doit embarquer l'environnement nécessaire à son fonctionnement.

Par exemple, on peut avoir :

sauvegarde_mysql.py       quelques Ko

sauvegarde_mysql.exe      plusieurs dizaines de Mo

La taille exacte dépend notamment de la version de Python et des bibliothèques utilisées.

Avec :

pyinstaller --onefile mon_script.py

tout est regroupé dans **un seul fichier `.exe`**, ce qui est très pratique pour la distribution, mais peut augmenter le temps de démarrage et la taille du fichier.

## Temps de démarrage

Avec `--onefile`, l'exécutable doit notamment préparer son environnement au démarrage.

Un programme très simple peut donc démarrer légèrement moins rapidement qu'un script Python exécuté directement.

Pour un outil lancé occasionnellement, comme un programme de sauvegarde, cet inconvénient est généralement négligeable.
-

## Dépendances externes

Un `.exe` PyInstaller **n'embarque pas nécessairement tout ce dont le programme a besoin à l'extérieur de Python**.

Par exemple, si le programme utilise :

mysql.exe

mysqldump.exe

ces programmes externes doivent toujours être disponibles.

Ainsi, un code comme :

MYSQLDUMP = r"C:\wamp64\bin\mysql\mysql9.5.0\bin\mysqldump.exe"

reste problématique sur un autre ordinateur si MySQL/WAMP n'est pas installé au même endroit.

Il faut donc distinguer les bibliothèques Python intégrées dans le .exe et les programmes externes qui doivent être présents sur le PC

# 9. Comparaison des deux solutions

||Script `.py`|Exécutable `.exe`|
|---|---|---|
|Python nécessaire|Oui|Non, en principe|
|Python Launcher nécessaire|Oui|Non|
|Installation des bibliothèques|Oui|Non, généralement|
|Double-clic|Oui, si association correcte|Oui|
|Taille|Très faible|Plus importante|
|Mise à jour|Remplacer le `.py`|Regénérer le `.exe`|
|Distribution|Moins pratique|Très pratique|
|Dépendances externes|À installer/configurer|Toujours nécessaires si utilisées|
|Développement|Très pratique|Nécessite une étape de génération|
|Portabilité|Dépend de l'environnement Python|Meilleure|
