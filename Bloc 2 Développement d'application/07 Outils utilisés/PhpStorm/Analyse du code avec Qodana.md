# 1. Présentation

**Qodana** est l'outil d'analyse de code de JetBrains.

Il permet d'effectuer une analyse globale du projet en utilisant les inspections JetBrains. Contrairement aux inspections affichées au fur et à mesure dans l'éditeur, Qodana permet d'obtenir une **analyse complète du projet** et de regrouper les problèmes détectés dans une vue dédiée.

Dans le projet, Qodana peut notamment analyser :

- PHP ;
- JavaScript ;
- HTML ;
- CSS ;
- JSON ;
- et d'autres fichiers utilisés par le projet.

L'analyse locale est particulièrement intéressante pour vérifier l'ensemble du projet avant une modification importante ou après une évolution de la configuration de contrôle du code.
# 2. Analyse locale

Qodana peut être exécuté directement depuis PhpStorm, sans avoir besoin de mettre en place une intégration CI/CD.

L'analyse locale permet notamment de :

- rechercher les problèmes dans l'ensemble du projet ;
- détecter les erreurs qui ne sont pas forcément visibles dans le fichier actuellement ouvert ;
- analyser le JavaScript avec les inspections configurées ;
- vérifier le code PHP ;
- identifier des problèmes de qualité ou de sécurité ;
- obtenir une vue globale des problèmes détectés.
# 3. Fichier de configuration Qodana et exclusions

La configuration de Qodana est stockée dans le fichier :

```text
qodana.yaml
```

Ce fichier doit être placé **à la racine du projet**, au même niveau que les principaux répertoires du projet.

Il permet notamment de définir :

- le profil d'analyse utilisé ;
- les inspections activées ou désactivées ;
- les répertoires ou fichiers à exclure ;
- la version de PHP utilisée ;
- le linter Qodana ;
- et éventuellement les conditions de qualité utilisées dans une chaîne CI/CD.
## Exclure les composants externes

Les bibliothèques externes intégrées au projet ne doivent généralement pas être analysées par Qodana. Leur code n'est en effet pas maintenu dans le projet et les avertissements détectés ne peuvent généralement pas être corrigés directement.

Par exemple, si les composants externes sont stockés dans :

```text
public/
└── composant/
    ├── tinymce/
    ├── bootstrap/
    └── ...
```

il est possible d'exclure ce répertoire de l'analyse.

Dans `qodana.yaml` :

```yaml
exclude:
  - name: All
    paths:
      - public/composant/
```

Qodana n'analysera alors pas les fichiers situés dans `public/composant/`.

Si le répertoire contient également du code développé dans le cadre du projet, il est préférable de ne pas exclure tout le répertoire. Il faut alors cibler uniquement les bibliothèques externes.

Par exemple :

```yaml
exclude:
  - name: All
    paths:
      - public/composant/tinymce/
      - public/composant/bootstrap/
```

Cette organisation permet de conserver l'analyse Qodana sur le code du projet tout en ignorant les bibliothèques tierces.
# 4. Lancer Qodana

Dans PhpStorm, ouvrir la fenêtre :

**Problems**

Puis sélectionner l'onglet :

**Qodana**

Si aucune analyse n'a encore été réalisée, utiliser l'action permettant de lancer une analyse locale, généralement accessible avec :

**Try Locally**

Selon la version de PhpStorm, l'accès peut également se trouver dans :

**Tools → Qodana**

Puis :

**Try Locally**

Qodana lance alors l'analyse du projet.

# 5. Résultats de l'analyse

Une fois l'analyse terminée, les problèmes sont affichés dans l'onglet **Qodana** de la fenêtre **Problems**.

Les problèmes peuvent notamment être classés par :

- fichier ;
- type de problème ;
- niveau de gravité ;
- inspection utilisée.

Un problème peut par exemple apparaître sous la forme :

```text
administration/classement/index.js

JSHint: Functions declared within loops...
```

avec des informations telles que :

```text
Severity: CRITICAL
Inspection: JSHint
Line: 120
Column: 28
```

Un clic sur le problème permet de revenir directement à l'emplacement concerné dans le fichier.

# 6. Relancer une analyse

Après une modification du code ou de la configuration, il est possible de relancer Qodana.

Dans :

**Problems → Qodana**

utiliser l'action de relance disponible dans la barre d'outils, généralement représentée par l'icône :

**↻**

Une autre possibilité consiste à utiliser :

**Tools → Qodana → Try Locally**

Cette opération permet de lancer une nouvelle analyse du projet.

# 7. Quand relancer Qodana ?

Il est particulièrement utile de relancer Qodana après :

- une modification du fichier `.jshintrc` ;
- une modification de la configuration des inspections ;
- une modification importante du code ;
- l'ajout d'un nouveau module ;
- une refactorisation importante ;
- la correction d'un ensemble de problèmes ;
- une mise à jour importante de PhpStorm ou des outils JetBrains.

Par exemple, après avoir ajouté :

```json
"-W083": true
```

dans `.jshintrc`, il est préférable de relancer l'analyse afin de vérifier que l'avertissement **W083** n'est plus remonté.

# 8. Qodana et JSHint

Qodana peut utiliser l'inspection **JSHint** pour analyser le JavaScript.

Un problème peut donc apparaître avec :

```text
Inspection: JSHint
```

Cela signifie que Qodana utilise l'inspection JSHint pour détecter ce problème.

Il est important de distinguer :

```text
JSHint
    ↓
Analyse du JavaScript selon ses règles

Qodana
    ↓
Orchestre les inspections sur l'ensemble du projet
```

Qodana ne se limite donc pas à JavaScript.

# 9. Configuration JSHint

Le projet utilise un fichier :

```text
.jshintrc
```

Ce fichier contient les règles utilisées par JSHint.

Par exemple :

```json
{
  "esversion": 12,
  "browser": true,
  "devel": true,
  "undef": true,
  "unused": true,
  "curly": true,
  "eqeqeq": true
}
```

Après toute modification de ce fichier, une nouvelle analyse Qodana doit être effectuée afin de vérifier les résultats.

# 10. Exemple avec une erreur JSHint

Supposons que Qodana affiche :

```text
JSHint: Functions declared within loops referencing
an outer scoped variable may lead to confusing semantics.

Inspection: JSHint
Warning: W083
```

Le problème peut être localisé dans :

```text
administration/classement/index.js
```

avec :

```text
Ligne : 120
Colonne : 28
```

Si la règle W083 a été volontairement désactivée dans `.jshintrc` :

```json
"-W083": true
```

il faut relancer Qodana pour vérifier que le problème n'est plus signalé.

# 11. Comprendre les niveaux de gravité

Qodana attribue un niveau de gravité aux problèmes détectés.

Selon la configuration utilisée, on peut notamment rencontrer :

|Niveau|Signification|
|---|---|
|`CRITICAL`|Problème considéré comme très important et nécessitant une attention particulière.|
|`HIGH`|Problème important susceptible d'avoir un impact significatif.|
|`MODERATE`|Problème de gravité intermédiaire.|
|`LOW`|Problème moins important ou amélioration possible.|
|`INFO`|Information ou suggestion d'amélioration.|

La gravité dépend de l'inspection et du profil Qodana utilisé.

# 12. Problèmes nouveaux et problèmes existants

Qodana peut également indiquer l'état d'un problème.

Par exemple :

```text
baselineState: NEW
```

signifie que le problème est considéré comme **nouveau** par rapport à la référence utilisée.

Cette distinction est particulièrement utile dans un projet existant comportant déjà plusieurs problèmes.

L'objectif peut alors être de :

1. conserver les anciens problèmes temporairement ;
    
2. éviter d'en introduire de nouveaux ;
    
3. corriger progressivement les problèmes existants.
    

# 13. Analyse après correction

Une bonne méthode de travail consiste à fonctionner ainsi :

```text
Modifier le code
       ↓
Lancer Qodana
       ↓
Examiner les problèmes
       ↓
Corriger
       ↓
Relancer Qodana
       ↓
Vérifier la disparition des problèmes
```

Il est préférable de ne pas chercher à corriger aveuglément tous les avertissements.

Certaines inspections peuvent ne pas être pertinentes pour le projet.

Dans ce cas, il est possible de :

- modifier la configuration ;
    
- désactiver une inspection ;
    
- modifier sa gravité ;
    
- utiliser une configuration spécifique au projet.
    
# 14. Qodana et les inspections PhpStorm

Il faut distinguer les deux usages.

### PhpStorm

Les inspections PhpStorm sont principalement utilisées pendant le développement.

Elles permettent d'obtenir immédiatement des indications dans le fichier en cours d'édition.

```text
Édition du fichier
       ↓
Inspection immédiate
       ↓
Correction
```

### Qodana

Qodana permet de réaliser une analyse plus globale :

```text
Projet complet
       ↓
Qodana
       ↓
Analyse des différents fichiers
       ↓
Rapport global
```

Les deux outils sont donc complémentaires.

# 15. Utilisation recommandée dans le projet

Pour ce projet, une utilisation simple peut être retenue :

### Pendant le développement

Utiliser les inspections intégrées de PhpStorm.

### Régulièrement

Lancer une analyse locale Qodana :

**Problems → Qodana → Try Locally**

### Après une modification de configuration

Relancer Qodana, notamment après une modification de :

```text
.jshintrc
```

ou de la configuration des inspections.

# 16. Qodana sans CI/CD

L'utilisation locale de Qodana ne nécessite pas de mettre immédiatement en place une chaîne CI/CD.

Il est donc possible de l'utiliser simplement comme outil d'analyse du projet :

```text
PhpStorm
   │
   ├── Développement
   │
   ├── Inspections
   │
   └── Qodana
         │
         └── Analyse locale
```

Une intégration CI/CD pourra éventuellement être ajoutée plus tard si le projet en a besoin.

# 17. Workflow conseillé

Le workflow retenu pour le projet peut donc être résumé ainsi :

```text
                 DÉVELOPPEMENT
                       │
                       ▼
                   PhpStorm
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
        Inspections          JSHint
        PhpStorm            .jshintrc
              │                 │
              └────────┬────────┘
                       │
                       ▼
                    Qodana
                       │
                       ▼
              Analyse globale
                  du projet
```

### Objectif

- **PhpStorm** : assistance immédiate pendant le développement ;
    
- **JSHint** : contrôle spécifique du JavaScript selon les règles du projet ;
    
- **Qodana** : analyse globale et centralisée du projet.
    

Cette organisation permet de conserver une configuration JSHint volontairement légère tout en laissant à Qodana et aux inspections PhpStorm les contrôles plus généraux.