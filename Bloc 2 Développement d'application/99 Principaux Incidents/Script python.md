Lorsque tu exécutes `pip install colorama`, Python installe parfois la bibliothèque `colorama` dans le répertoire de ton utilisateur plutôt que dans `C:\Program Files`. Ce comportement est normal et volontaire sous Windows.

## 1. Pourquoi Python installe-t-il les bibliothèques dans ton répertoire utilisateur ?

Il y a plusieurs raisons :

- Les droits d'administrateur : le dossier `C:\Program Files` est un répertoire protégé par Windows. Pour y installer des fichiers, il faut généralement des droits d'administrateur.
    
- La sécurité : installer des bibliothèques dans ton répertoire personnel évite de modifier l'installation globale de Python et limite les risques de conflit avec d'autres programmes.
    
- L'installation utilisateur de Python : si Python est configuré pour une installation utilisateur ou si `pip` ne peut pas écrire dans le répertoire global, il peut installer les bibliothèques dans un dossier tel que :
    
`C:\Users\TonNom\AppData\Roaming\Python\Python313\site-packages`

## 2. Comment éviter cette installation dans le répertoire utilisateur ?


Option 1 : Installer la bibliothèque globalement

```batch
python -m pip install colorama
```

Si l'installation de Python se trouve dans `C:\Program Files`, pip pourra généralement installer `colorama` dans son dossier global `Lib\site-packages`, à condition que les permissions le permettent.

Attention : cela ne fonctionnera pas nécessairement si Python est configuré comme une installation gérée ou si son environnement est protégé.

Option 2 : Choisir un répertoire précis avec `--target` :

```
python -m pip install colorama --target "C:\Program Files\Python313\Lib\site-packages"
```

Option 3 : Utiliser un environnement virtuel

Recommandé pour les projets Python

Place-toi dans le dossier de ton projet et exécute :

```
python -m venv .venv
.venv\Scripts\activate
python -m pip install colorama
```

La bibliothèque sera installée dans le dossier `.venv` de ton projet, ce qui permet d'isoler ses dépendances des autres projets et de l'installation globale de Python.

## 3. Comment vérifier où colorama est installé ?


```
python -m pip show colorama
```

```
python -c "import colorama; print(colorama.__file__)"
```

Cela permet de connaître le chemin exact du module que Python utilise.

