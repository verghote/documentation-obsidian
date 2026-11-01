

Lorsque l'on parle de **Composer**, il est souvent présenté comme un gestionnaire de dépendances permettant d'installer des bibliothèques tierces. C'est effectivement sa fonction principale, mais ce n'est pas la seule.

Il est tout à fait possible d'utiliser Composer **sans installer le moindre package externe**. Dans ce cas, Composer sert uniquement à gérer le chargement automatique des classes (**autoloading**) et les espaces de noms (**namespaces**) de l'application.

Cette utilisation est particulièrement intéressante pour les projets pédagogiques, les petits développements ou les applications métiers développées entièrement en interne.

## Pourquoi utiliser Composer dans ce cas ?

Dans un projet PHP, les classes sont généralement réparties dans plusieurs fichiers.

Sans mécanisme d'autoload, il est nécessaire d'inclure chaque fichier manuellement.

```php
require_once '../src/ClasseTechnique/Erreur.php';
require_once '../src/ClasseMetier/Produit.php';
require_once '../src/ClasseMetier/Client.php';
```

Cette approche présente plusieurs inconvénients :

- le nombre de `require` augmente avec la taille du projet ;
- les oublis sont fréquents ;
- le code devient difficile à maintenir ;
- un déplacement de fichier oblige à modifier plusieurs parties du projet.

Pour éviter cela, de nombreux développeurs écrivent leur propre autoloader avec `spl_autoload_register()`.

Par exemple :

```php
spl_autoload_register(function ($classe) {

    $prefixes = [
        'ClasseTechnique\\' => __DIR__ . '/../src/ClasseTechnique/',
        'ClasseMetier\\'    => __DIR__ . '/../src/ClasseMetier/'
    ];

    foreach ($prefixes as $prefix => $repertoire) {

        if (str_starts_with($classe, $prefix)) {

            $fichier = $repertoire . substr($classe, strlen($prefix)) . '.php';

            if (is_file($fichier)) {
                require $fichier;
            }
        }
    }

});
```

Cette solution fonctionne correctement, mais elle présente plusieurs limites :

- elle doit être développée et testée ;
- elle doit évoluer avec le projet ;
- elle ne respecte pas toujours les recommandations PSR ;
- elle devient rapidement complexe lorsque plusieurs espaces de noms sont utilisés.

Composer permet de remplacer complètement cet autoloader personnalisé.

# Composer comme simple autoloader

Dans cette approche, Composer n'est utilisé que pour deux tâches :

- gérer les espaces de noms (namespaces) ;
- générer automatiquement l'autoloader.

Aucune bibliothèque externe n'est installée.

Le fichier `composer.json` contient uniquement la configuration de l'autoload.

Exemple :

```json
{  
  "autoload": {  
    "psr-4": {  
      "ClasseTechnique\\": "src/ClasseTechnique/",  
      "ClasseMetier\\": "src/ClasseMetier/"  
    }  
  }  
}
```

Dans cet exemple :

- toutes les classes appartenant au namespace `ClasseTechnique` seront recherchées automatiquement dans le dossier `src\ClasseTechnique`.
- toutes les classes appartenant au namespace `ClasseMetier` seront recherchées automatiquement dans le dossier `src\ClasseMetier`.
# Organisation du projet

Une organisation classique peut être la suivante :

```text
Projet/
│
├── composer.json
├── public/
│   └── index.php
│
├── src/
│   ├── ClasseTechnique/
│   ├── ClasseMetier/
│
└── vendor/
```

Le dossier `vendor` est créé automatiquement par Composer.

Il contient notamment :

```text
vendor/
│
├── autoload.php
└── composer/
```

Le fichier important est :

```text
vendor/autoload.php
```

C'est lui qui sera inclus au démarrage de l'application.

# Générer l'autoloader

Une fois le fichier `composer.json` créé, il suffit d'exécuter :

```bash
composer dump-autoload
```

Composer analyse la configuration puis génère automatiquement l'autoloader.

Cette commande peut être relancée autant de fois que nécessaire, par exemple après :

- l'ajout d'une nouvelle classe ;
- le déplacement d'un fichier ;
- la création d'un nouveau namespace.

# Charger l'autoloader

Dans le point d'entrée de l'application (`index.php`, par exemple), un seul `require` est nécessaire.

```php
require __DIR__ . '/../vendor/autoload.php';
```

À partir de ce moment, toutes les classes déclarées dans les namespaces configurés sont chargées automatiquement.

# Exemple complet

Considérons l'arborescence suivante :

```text
Projet/
│
├── composer.json
│
├── public/
│   └── index.php
│
└── src/
    └── ClasseTechnique/
        └── Erreur.php
    ├── ClasseMetier/
      └── categorie.php
```

Le fichier `composer.json` contient :

```json
{  
  "autoload": {  
    "psr-4": {  
      "ClasseTechnique\\": "src/ClasseTechnique/",  
      "ClasseMetier\\": "src/ClasseMetier/"  
    }  
  }  
}
```

La classe :

```php
<?php

namespace ClasseMetier;  
  
use ClasseTechnique\Select;  
  
// définition de la table catégorie : id, nom, ageMin, ageMax  
  
class Categorie  
{  
    /**  
     * retourne les catégories en y ajoutant l'intervalle des dates de naissance pour les catégories     * @return array  
     */  
    public static function getAll(): array  
    { ... }
}
```

Le point d'entrée :

```php
<?php

use ClasseMetier\Categorie;  
use ClasseTechnique\Page;  
  
require $_SERVER['DOCUMENT_ROOT'] . "/../bootstrap/autoload.php";  
  
// alimentation et affichage de l'interface  
// chargement des catégories et du composant assurant la génération d'un document PDF à partir de la page HTML  
$page = new Page();  
$page->setTitre("Liste des catégories")  
    ->setDonnee("lesCategories", Categorie::getAll())  
    ->addScript("/composant/html2pdf/html2pdf.bundle.min.js")  
    ->afficher();
```

Aucun autre `require` n'est nécessaire.

Composer localise automatiquement le fichier `Categorie.php`, l'inclut puis instancie la classe.
Composer localise automatiquement le fichier `Page.php`, l'inclut puis instancie la classe.

# Que fait réellement Composer ?

Lorsque PHP rencontre l'instruction :

```php
new Page();
```

il ne connaît pas encore cette classe.

L'autoloader de Composer est alors appelé.

Il effectue les opérations suivantes :

1. il récupère le namespace de la classe ;
2. il recherche le préfixe correspondant dans `composer.json` ;
3. il construit le chemin du fichier ;
4. il charge automatiquement ce fichier ;
5. PHP poursuit ensuite l'exécution du programme.

Le développeur n'a donc plus à gérer lui-même les inclusions de fichiers.

# Préparer un projet pour l'avenir

Même si une application ne dépend aujourd'hui d'aucune bibliothèque externe, utiliser Composer présente un autre avantage : le projet est immédiatement prêt à évoluer.

Par exemple, si l'on souhaite ultérieurement utiliser la bibliothèque **Monolog**, il suffira d'exécuter :

```bash
composer require monolog/monolog
```

L'autoloader est déjà présent et aucune modification de l'application n'est nécessaire.

Le projet suit ainsi les standards actuels de l'écosystème PHP.

# Bonnes pratiques

Lorsque Composer est utilisé uniquement pour son autoloader, il est recommandé de :

- respecter la norme PSR-4 pour l'organisation des dossiers ;
- ne jamais modifier le contenu du dossier `vendor` ;
- exécuter `composer dump-autoload` après une modification importante de l'arborescence des classes ;
- conserver le fichier `composer.json` dans le système de gestion de versions (Git).

# À retenir

> **Composer n'est pas uniquement un gestionnaire de dépendances.**

Même sans installer de bibliothèque externe, il permet de :

- gérer automatiquement les espaces de noms ;
- générer un autoloader conforme à la norme PSR-4 ;
- supprimer les `require` manuels ;
- remplacer un autoloader développé avec `spl_autoload_register()` ;
- préparer le projet à accueillir facilement des dépendances dans le futur.

Cette approche est aujourd'hui considérée comme une bonne pratique pour tout nouveau projet PHP, quelle que soit sa taille.