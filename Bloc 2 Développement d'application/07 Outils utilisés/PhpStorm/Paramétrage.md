PhpStorm distingue deux catégories de paramètres :

- les **paramètres globaux**, communs à tous les projets de l'utilisateur ;
- les **paramètres spécifiques à un projet**, enregistrés dans le dossier `.idea`.

Cette distinction permet de personnaliser l'environnement de développement tout en conservant, pour chaque projet, les réglages qui lui sont propres.

# Les paramètres globaux

Les paramètres globaux sont enregistrés dans le profil de l'utilisateur et sont automatiquement disponibles pour tous les projets ouverts dans PhpStorm.

Ils ne sont **pas** stockés dans le dossier `.idea`.

Parmi les principaux paramètres globaux, on trouve :

- les interpréteurs PHP (PHP 8.2, PHP 8.3, etc.) ;
- les thèmes de l'IDE ;
- les polices de caractères ;
- les raccourcis clavier ;
- les plugins installés ;
- les paramètres de l'éditeur (indentation, coloration syntaxique, etc.) ;
- les outils externes ;
- les serveurs de débogage.

> **Remarque**
>
> Lorsqu'un interpréteur PHP est configuré une première fois dans PhpStorm, il est ensuite disponible pour tous les projets. Il n'est donc pas nécessaire de le redéfinir à chaque nouveau projet.

# Les paramètres spécifiques au projet

Chaque projet possède un dossier nommé `.idea`.

Ce dossier contient les paramètres propres au projet courant.

Parmi les informations enregistrées, on trouve notamment :

- les modules du projet ;
- les connexions aux bases de données ;
- définir la racine web (dossier public) :  clic droit sur le dossier public  → Mark Directory As   → Resource Root
- les configurations d'exécution (Run Configurations) ;
- les profils d'inspection du code ;
- les styles de codage spécifiques au projet ;
- certaines informations relatives au contrôle de versions (Git).

Contrairement aux paramètres globaux, ces informations ne concernent qu'un seul projet.

# Les connexions aux bases de données

Les connexions définies dans l'outil **Database** sont enregistrées dans le dossier `.idea`.

Elles sont principalement stockées dans les fichiers :

- `dataSources.xml`
- `dataSources.local.xml`

Ces fichiers permettent de retrouver automatiquement les connexions à la réouverture du projet.

> **Remarque**
>
> Les informations sensibles (par exemple les mots de passe) sont généralement stockées dans des fichiers locaux qui ne doivent pas être partagés.

---

# Pourquoi conserver le dossier `.idea` ?

Lorsqu'un projet est mis à jour à l'aide de Git, il est souvent préférable de conserver le dossier `.idea`.

L'étudiant retrouve ainsi immédiatement :

- ses connexions aux bases de données ;
- ses configurations d'exécution ;
- son organisation de travail dans PhpStorm.

Le projet est remis dans son état d'origine tout en conservant le confort de travail de l'utilisateur.

# Résumé

| Paramètres globaux | Paramètres du projet (`.idea`) |
|--------------------|--------------------------------|
| Interpréteurs PHP | Connexions aux bases de données |
| Thèmes | Configurations d'exécution |
| Raccourcis clavier | Styles de codage du projet |
| Plugins | Profils d'inspection |
| Paramètres de l'éditeur | Paramètres Git du projet |

Les paramètres globaux sont communs à tous les projets tandis que le dossier `.idea` contient uniquement les informations propres au projet courant.