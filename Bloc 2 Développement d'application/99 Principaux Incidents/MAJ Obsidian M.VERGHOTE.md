
git pull :
error: cannot stat 'Bloc 2 Développement d'application/02 Développement Web/03 Éléments pratiques/Classe technique/03 Les classes gérant les fichiers/Les classes Input/Utilisation des classes InputFile et InputFileImg.md': Filename too long


**Solution :** Activez le support des chemins longs dans Git en exécutant cette commande dans votre terminal (en tant qu'administrateur si nécessaire) :

Bash

```
git config --global core.longpaths true
```

Supprimer le fichier local et accepter la version distante (Le plus simple)

```
git clean -f "Bloc 2 Développement d'application/01 Présentation/02 - Bloc 2 (Support et mise à disposition de services informatiques).md"
```



error: The following untracked working tree files would be overwritten by merge:

        Bloc 2 Développement d'application/01 Présentation/02 - Bloc 2 (Support et mise à disposition de services informatiques).md

        Bloc 2 Développement d'application/01 Présentation/03 - Bloc 3 (Cybersécurité des services informatiques).md

Voici la cause et les solutions pour chacun de ces deux problèmes Git :

### 1. Erreur « Filename too long »

Le chemin de votre fichier dépasse la limite par défaut imposée par Windows ou Git (260 caractères).

**Solution :** Activez le support des chemins longs dans Git en exécutant cette commande dans votre terminal (en tant qu'administrateur si nécessaire) :

Bash

```
git config --global core.longpaths true
```

> **Note :** Si vous utilisez Windows, assurez-vous également que la limite globale est désactivée dans le système (via l'Éditeur de stratégie de groupe ou le Registre Windows si la commande Git ne suffit pas).

### 2. Erreur « Untracked working tree files would be overwritten by merge »

Git refuse d'effectuer le merge (ou le pull) car vous avez des fichiers locaux non suivis (_untracked_) qui portent exactement le même nom et emplacement que des fichiers provenant de la branche distante.

**Solutions au choix :**

- **Option A : Conserver vos fichiers locaux (les sauvegarder)** Déplacez ou renommez temporairement le dossier `Bloc 2 Développement d'application` hors de votre dépôt, effectuez le merge, puis réintégrez vos modifications si besoin.
    
- **Option B : Écraser vos fichiers locaux avec la version distante** Si vous n'avez pas besoin de conserver ces fichiers locaux :
    
    Bash
    
    ```
    git clean -fd
    ```
    
    _(Attention : cette commande supprime définitivement tous les fichiers et dossiers non suivis)._
    
- **Option C : Les ajouter à Git avant de merger** Si vous souhaitez conserver ces fichiers et les fusionner :
    
    Bash
    
    ```
    git add .
    git commit -m "Sauvegarde locale avant merge"
    git merge <nom-de-la-branche>
    ```
    

Une fois l'option choisie exécutée, relancez votre commande `git merge` ou `git pull`.

Updating 50a85ac..fc2ace4

error: The following untracked working tree files would be overwritten by merge:

        Bloc 2 Développement d'application/01 Présentation/02 - Bloc 2 (Support et mise à disposition de services informatiques).md

Ce problème survient à nouveau car vous avez toujours un fichier local non suivi (_untracked_) qui entre en conflit avec la version distante venant d'être rapatriée par le merge / pull.

Pour débloquer la situation, choisissez l'une des trois méthodes selon ce que vous souhaitez faire de votre fichier local :

### Option 1 : Supprimer le fichier local et accepter la version distante (Le plus simple)

Si vous ne tenez pas à conserver le contenu local de ce fichier :

Bash

```
git clean -f "Bloc 2 Développement d'application/01 Présentation/02 - Bloc 2 (Support et mise à disposition de services informatiques).md"
```

_Ou pour nettoyer tous les fichiers non suivis d'un coup :_ `git clean -fd`

### Option 2 : Sauvegarder le fichier avant de merger

Si vous voulez garder votre travail local de côté :

1. Déplacez ou renommez temporairement ce fichier hors de votre dossier Git.
    
2. Relancez votre `git pull` ou `git merge`.
    
3. Comparez et réintégrez vos notes manuelles si besoin.
    

### Option 3 : Intégrer le fichier dans votre historique Git

Si vous souhaitez conserver votre version locale et la fusionner proprement :

Bash

```
git add .
git commit -m "Sauvegarde du fichier de présentation local"
git pull
```

Une fois l'une de ces étapes réalisée, relancez votre commande initiale (`git pull` ou `git merge`).