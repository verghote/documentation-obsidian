# Présentation générale

La classe `FileManagerFactory` a pour rôle de centraliser la création des gestionnaires de fichiers utilisés dans l'application.

Elle évite de dupliquer dans les services la logique nécessaire à la création des objets :

- `ImageManager` pour les fichiers images ;
- `PdfManager` pour les fichiers PDF et documents.

Chaque gestionnaire de fichiers nécessite plusieurs paramètres de configuration :

- le répertoire physique de stockage ;
- la taille maximale autorisée ;
- les règles de transformation du nom ;
- les options spécifiques au type de fichier.

Ces paramètres sont chargés automatiquement depuis les fichiers de configuration associés.

La classe `FileManagerFactory` garantit donc que tous les gestionnaires de fichiers sont créés avec une configuration homogène.

# Principe d'utilisation

Les services métier ne créent jamais directement un `ImageManager` ou un `PdfManager`.

Ils utilisent uniquement la fabrique.

Exemple :

```php
$this->imageManager = FileManagerFactory::club();
````

Le service `ServiceClub` ne connait donc pas :

- le répertoire physique utilisé ;
- les extensions autorisées ;
- les dimensions maximales ;
- les règles de traitement des images.

Il demande simplement :

> "Donne-moi le gestionnaire permettant de gérer les logos des clubs."

# Les méthodes publiques

## Gestion des fichiers PDF

```php
public static function pdf(): PdfManager
```

Retourne un gestionnaire configuré pour les fichiers PDF génériques.

Le répertoire et les paramètres sont chargés depuis la configuration :

```
config/pdf.php
```

## Gestion des documents

```
public static function document(): PdfManager
```

Retourne un gestionnaire destiné aux documents PDF stockés dans un répertoire spécifique.

Configuration utilisée :

```
config/document.php
```

## Gestion des images

```
public static function image(): ImageManager
```

Retourne un gestionnaire d'images générique.

Configuration utilisée :

```
config/image.php
```

## Gestion des logos des clubs

```
public static function club(): ImageManager
```

Retourne un gestionnaire d'images spécialisé pour les logos des clubs.

Configuration utilisée :

```
config/club.php
```

Exemple :

```
[
    'maxSize' => 500000,
    'sansAccent' => true,
    'redimensionner' => true,
    'width' => 350,
    'height' => 350
]
```

# Chargement de la configuration

La création d'un gestionnaire passe toujours par une méthode privée dédiée.

Exemple :

```php
private static function createImageManager(string $config, string $repertoire): ImageManager
```

Cette méthode :

1. charge le fichier de configuration ;
2. récupère les paramètres disponibles ;
3. applique des valeurs par défaut si nécessaire ;
4. construit l'objet `ImageManager`.

Exemple :

```php
$parametres = Config::chargerPhp($config);

return new ImageManager(
    repertoire: $repertoire,
    maxSize: $parametres['maxSize'] ?? 0,
    sansAccent: $parametres['sansAccent'] ?? true,
    redimensionner: $parametres['redimensionner'] ?? false,
    width: $parametres['width'] ?? 0,
    height: $parametres['height'] ?? 0
);
```

# Pourquoi `rename` n'est pas un paramètre du constructeur ?

Le renommage automatique en cas de fichier existant n'est volontairement pas une propriété permanente du `FileManager`.

Cette décision est importante car le renommage dépend de l'opération réalisée.

Le gestionnaire de fichiers doit savoir **comment manipuler un fichier**, mais il ne doit pas décider **dans quel contexte métier il est utilisé**.

## Exemple : ajout d'un logo

Lorsqu'un nouveau club est créé :

```
Club A
logo.png
```

puis :

```
Club B
logo.png
```

Les deux fichiers doivent pouvoir coexister.

Le service demande alors un ajout avec génération d'un nom unique.

Résultat :

```
logo.png
logo(1).png
```

## Exemple : remplacement d'un logo

Lorsqu'un club possède déjà :

```
logo.png
```

et que l'utilisateur charge un nouveau fichier, le comportement attendu est :

```
ancien logo.png
        |
        ▼
nouveau contenu
        |
        ▼
logo.png
```

Le nom doit rester identique.

Un renommage automatique produirait :

```
logo.png
logo(1).png
```

ce qui créerait un nouveau fichier au lieu de remplacer l'ancien.

# Conséquence architecturale

Le choix du renommage appartient donc au scénario métier.

Il est géré au niveau des méthodes d'utilisation :

## Création d'un fichier

```
$imageManager->ajouterAvecNomUnique($fichier);
```

ou équivalent :

```
$imageManager->ajouter($fichier, true);
```

Le service indique :

> Le fichier doit être ajouté et un nom unique doit être généré si nécessaire.

## Remplacement

```php
$imageManager->remplacer($ancienNom, $fichier);
```

Le service indique :

> Le fichier existant doit conserver son nom.

# Pourquoi ne pas mettre cette option dans la configuration ?

Une configuration telle que :

```
[
    'rename' => true
]
```

semblerait plus simple mais elle introduirait une ambiguïté.

Le même gestionnaire pourrait être utilisé pour :

- créer un nouveau logo ;
- remplacer un logo ;
- restaurer un fichier ;
- effectuer une migration.

Ces opérations peuvent nécessiter des comportements différents.

Le renommage n'est donc pas une caractéristique du stockage.

C'est une décision liée à l'action réalisée.

# Responsabilités de `FileManagerFactory`

La classe respecte une séparation claire :

| Classe               | Responsabilité                                           |
| -------------------- | -------------------------------------------------------- |
| `FileManagerFactory` | Créer un gestionnaire correctement configuré             |
| `ImageManager`       | Gérer techniquement les fichiers images                  |
| `PdfManager`         | Gérer techniquement les fichiers PDF                     |
| Service métier       | Décider quand ajouter, remplacer ou supprimer un fichier |

# Résumé

`FileManagerFactory` fournit une création centralisée et sécurisée des gestionnaires de fichiers.

Elle permet :

- d'éviter la duplication de configuration ;
- de garantir une configuration homogène ;
- de masquer les détails techniques aux services métier ;
- de conserver une séparation claire entre stockage physique et logique métier.

Le choix de ne pas intégrer `rename` dans la configuration permet de conserver une architecture plus souple :

- la configuration décrit les caractéristiques permanentes du stockage ;
- le service métier décide du comportement nécessaire pour chaque opération.