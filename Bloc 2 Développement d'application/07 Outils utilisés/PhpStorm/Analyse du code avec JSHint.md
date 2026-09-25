
**JSHint** est un outil d'analyse statique pour **JavaScript**. Son rôle est d'analyser ton code **sans l'exécuter** afin de détecter des erreurs potentielles, des incohérences ou des mauvaises pratiques.

Il peut notamment signaler :

- des variables utilisées mais non déclarées ;
- des variables déclarées mais jamais utilisées ;
- des erreurs de syntaxe ;
- des `==` utilisés au lieu de `===` selon la configuration ;
- des variables globales involontaires ;
- des blocs ou instructions potentiellement problématiques ;
- certaines constructions JavaScript déconseillées.

JSHint peut être intéressant pour vérifier automatiquement tes fichiers `.js` et repérer certaines erreurs avant même de lancer ton application.

Il existe aujourd'hui des outils plus modernes, notamment **ESLint**, qui est beaucoup plus configurable mais qui nécessite l'intégration dans le projet de nombreux fichiers placés dans le répertoire node_modules.

Le fichier **`.jshintrc`** est un fichier de configuration utilisé par **JSHint**, un outil d’analyse statique pour JavaScript.

Il sert à définir **les règles de qualité et de style du code JavaScript** que l’on veut appliquer dans un projet.

Depuis les versions récentes de **PhpStorm**, JSHint n'est plus embarqué directement dans l'IDE : il utilise le **package JSHint installé avec Node.js**.

Le fichier `.jshintrc` contient uniquement les règles d'analyse. Il ne contient pas l'outil JSHint lui-même.

# 1. À quoi sert `.jshintrc`

Le fichier `.jshintrc` permet de configurer JSHint, afin d'adapter les règles selon le contexte du projet.
## Exemple simple de `.jshintrc`

```json
{
    "esversion": 6,
    "undef": true,
    "unused": true,
    "eqeqeq": true,
    "browser": true,
    "node": false
}
```

## Interprétation

- `undef: true`  
    → interdit l’utilisation de variables non déclarées.
- `unused: true`  
    → détecte les variables déclarées mais jamais utilisées.
- `eqeqeq: true`  
    → impose l’utilisation de `===` au lieu de `==`.
- `browser: true`  
    → autorise les objets du navigateur comme `window`, `document`, `alert`, etc.
- `esversion: 6`  
    → active la compréhension des fonctionnalités JavaScript ES6.
- `node: false`  
    → indique que le code est destiné au navigateur et non à Node.js.

# 2. Installation de JSHint avec Node.js

## Pourquoi installer Node.js ?

Le fichier `.jshintrc` ne suffit pas à exécuter une analyse.

PhpStorm utilise maintenant le package **JSHint fourni par npm**.  
Il faut donc installer JSHint avec Node.js.

Vérifier d'abord que Node.js est installé :

```bash
node -v
npm -v
```

Puis installer JSHint globalement :

```bash
npm install -g jshint
```

L'option `-g` signifie que JSHint est installé globalement sur la machine.

Les fichiers sont stockés dans le profil de l'utilisateur :
```
C:\Users\nomUser\AppData\Roaming\npm\node_modules\jshint
```

Cette installation :

- n'ajoute aucun fichier dans le projet ;
- ne crée pas de `node_modules`  (Aucun fichier Node n'est nécessaire dans les projets.);
- permet à plusieurs projets d'utiliser le même JSHint.

Pour vérifier l'installation :

```bash
jshint --version
```

# 3. Configuration de JSHint dans PhpStorm

PhpStorm doit connaître l'emplacement du package JSHint.

## Étape 1 : ouvrir la configuration

Dans PhpStorm :

```
Settings
→ Languages & Frameworks
→ JavaScript
→ Code Quality Tools
→ JSHint
```

## Étape 2 : activer JSHint

- cocher **Enable**
    
- renseigner le champ **JSHint package** : C:\Users\nomUser\AppData\Roaming\npm\node_modules\jshint

Le chemin peut être obtenu avec :

```bash
npm root -g
```

- cocher **Use config files**
- cocher **Custom configuration file**
- Indiquer le chemin vers le fichier .jshintrc : **J:\VirtualHostSlam\.jshintrc**

Le fichier `.jshintrc` peut donc être placé dans un dossier commun à plusieurs projets.
Le fichier `J:\VirtualHostSlam\.jshintrc` contient les règles communes ;
Tous les projets situés dans les sous-dossiers héritent automatiquement de cette configuration ;
Il n'est pas nécessaire de copier un fichier `.jshintrc` dans chaque projet.

Cette organisation permet d'appliquer les mêmes règles de qualité JavaScript à l'ensemble des projets de formation.
# 4. Utilisation du fichier `.jshintrc`

Une fois JSHint configuré dans PhpStorm les erreurs et avertissements concernant le fichier js actuellement ouvert  sont directement affichées dans la fenêtre Problems (ALT 6).

Exemple : JSHint: Missing semicolon. (W033)
# 5. Visualiser l'ensemble des erreurs

PhpStorm permet de lancer les inspections sur tous les fichiers du projet ou sur un périmètre particulier.

Pour obtenir une analyse de tout le projet : **Code → Inspect Code...** puis sélectionner **Whole project**.
Ou
depuis la fenêtre _Problems_ onglet **Project Errors**, cliquer sur le lien Inspect Code

Remarque  : PhpStorm possède **ses propres inspections JavaScript**, indépendamment de JSHint, la fenêtre affichera l'ensemble des erreurs du projet pas seulement celles indiquées pas JsHint.

# 6. Portée du paramétrage dans PhpStorm

PhpStorm conserve une partie de sa configuration dans le dossier :

```
.idea/
```

Ce dossier contient les paramètres liés à l'environnement de développement :

- inspections activées ;
- configuration de l'IDE ;
- réglages spécifiques au projet.

Cependant :

- `.idea` configure **PhpStorm** ;
- `.jshintrc` définit **les règles d'analyse JSHint**.

Le fichier `.jshintrc` peut être :

- placé dans un projet pour définir des règles spécifiques ;
- placé dans un dossier parent pour partager les règles entre plusieurs projets.

Dans l'environnement de formation, le fichier :

```
J:\VirtualHostSlam\.jshintrc
```

sert donc de configuration commune à tous les projets présents dans ce dossier.



# 7. Fonctionnement général

Le fonctionnement est donc :

```
Fichiers JavaScript
        │
        ▼
 Recherche du .jshintrc
        │
        ▼
 Règles communes J:\VirtualHostSlam\.jshintrc
        │
        ▼
      JSHint
        │
        ▼
 Analyse dans PhpStorm
        │
        ▼
 Erreurs et avertissements affichés
```


# 8. Conclusion

- `.jshintrc` sert à définir les règles de qualité JavaScript.
- JSHint est un outil d'analyse statique du code JavaScript.
- PhpStorm utilise désormais le package JSHint installé avec Node.js.
- Node.js est donc nécessaire pour installer et exécuter JSHint.
- L'installation globale avec :

```
npm install -g jshint
```

permet de conserver des projets légers.

- Le fichier `.jshintrc` peut être centralisé dans un dossier commun afin que plusieurs projets utilisent les mêmes règles.
- Dans l'environnement de formation, `J:\VirtualHostSlam\.jshintrc` constitue la configuration JSHint partagée par tous les projets.

