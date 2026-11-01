## 1. Introduction : Qu'est-ce que Swagger / OpenAPI ?

Lorsqu'on développe une API REST en PHP (comme notre API de calendrier ou GitHub), la question de sa documentation et de son test se pose rapidement.

- **OpenAPI :** C'est une spécification standard (un format de description d'API au format JSON ou YAML) qui décrit les routes disponibles, les paramètres attendus, et les structures de données renvoyées.
    
- **Swagger :** C'est l'ensemble des outils (comme Swagger UI) qui permettent de lire cette spécification OpenAPI pour générer une interface graphique web interactive et élégante.
    
- **L'avantage majeur :** Fini les documentations écrites à la main sur un fichier Word qui deviennent obsolètes dès que le code change. Grâce à des annotations directement placées dans le code PHP, la documentation est générée automatiquement.
    

## 2. Le package `zircote/swagger-php`

Pour faire le lien entre notre code PHP et le standard OpenAPI, on utilise la bibliothèque open-source `zircote/swagger-php`. Elle permet d'analyser (scanner) les commentaires de documentation (DocBlocks) et les attributs PHP de vos classes pour produire le fichier de spécification JSON attendu par l'interface Swagger UI.

## 3. Mise en place pas à pas

### Étape 1 : Installation via Composer

Dans le dossier racine de votre projet PHP, installez le paquet via Composer :

Bash

```
composer require zircote/swagger-php
```

### Étape 2 : Annoter les contrôleurs avec les attributs PHP

Dans votre architecture orientée objet, vos classes de contrôle gèrent les routes entrantes. En utilisant les attributs PHP de la bibliothèque (namespace `OpenApi\Attributes as OA`), vous documentez directement vos méthodes.

Voici un exemple concret appliqué à notre classe `CalendrierControleur` :

PHP

```
<?php
declare(strict_types=1);

namespace ClasseMetier;

use ClasseTechnique\Controleur;
use ClasseTechnique\ReponseHTTP;
use ClasseTechnique\RequeteHTTP;
use OpenApi\Attributes as OA;

/**
 * Classe Contrôleur : CalendrierControleur
 * Joue le rôle d’intermédiaire entre la requête HTTP et la logique métier.
 */
class CalendrierControleur extends Controleur
{
    /**
     * Retourne la liste des épreuves à venir.
     */
    #[OA\Get(
        path: '/', operationId: 'getAll', summary: 'Liste des épreuves à venir', tags: ['Calendrier'],
        responses: [
            new OA\Response(response: 200, description: 'Liste des épreuves', content: new OA\JsonContent(
                properties: [new OA\Property(property: 'data', type: 'array', items: new OA\Items(ref: '#/components/schemas/Epreuve'))]
            )),
            new OA\Response(ref: '#/components/responses/ErreurServeur', response: 500),
        ]
    )]
    public static function getAll(): void
    {
        self::executer(function () {
            $service = new CalendrierService();
            return $service->getAll();
        });
    }

    /**
     * Point d'entrée pour récupérer une épreuve par son numéro FFA.
     */
    #[OA\Get(
        path: '/{id}', operationId: 'getById', summary: 'Détail d\'une épreuve', tags: ['Calendrier'],
        parameters: [new OA\Parameter(ref: '#/components/parameters/IdFfa')],
        responses: [
            new OA\Response(response: 200, description: 'L\'épreuve', content: new OA\JsonContent(
                properties: [new OA\Property(property: 'data', ref: '#/components/schemas/Epreuve')]
            )),
            new OA\Response(ref: '#/components/responses/NonTrouve', response: 404),
            new OA\Response(ref: '#/components/responses/ErreurServeur', response: 500),
        ]
    )]
    public static function getById(string $id): void
    {
        self::executer(function () use ($id) {
            $service = new CalendrierService();
            return $service->getById($id);
        });
    }

    /**
     * Point d'entrée pour ajouter une nouvelle épreuve (code 201 en cas de succès).
     */
    #[OA\Post(
        path: '/', operationId: 'ajouter', summary: 'Ajoute une épreuve', tags: ['Calendrier'],
        security: [['bearerAuth' => []]],
        requestBody: new OA\RequestBody(required: true, content: new OA\JsonContent(ref: '#/components/schemas/Epreuve')),
        responses: [
            new OA\Response(response: 201, description: 'Épreuve créée', content: new OA\JsonContent(
                properties: [new OA\Property(property: 'data', ref: '#/components/schemas/Epreuve')]
            )),
            new OA\Response(ref: '#/components/responses/RequeteInvalide', response: 400),
            new OA\Response(ref: '#/components/responses/NonAuthentifie', response: 401),
            new OA\Response(ref: '#/components/responses/Conflit', response: 409),
            new OA\Response(ref: '#/components/responses/DonneesInvalides', response: 422),
            new OA\Response(ref: '#/components/responses/ErreurServeur', response: 500),
        ]
    )]
    public static function ajouter(): void
    {
        self::executer(function () {
            $data = RequeteHTTP::getData();
            if (empty($data)) {
                ReponseHTTP::sendError("Aucune donnée fournie pour l'ajout", 400);
            }
            $service = new CalendrierService();
            return $service->ajouter($data);
        }, 201);
    }

    /**
     * Point d'entrée pour modifier une épreuve existante.
     */
    #[OA\Patch(
        path: '/{id}', operationId: 'modifier', summary: 'Modifie partiellement une épreuve (id non modifiable)', tags: ['Calendrier'],
        security: [['bearerAuth' => []]],
        parameters: [new OA\Parameter(ref: '#/components/parameters/IdFfa')],
        requestBody: new OA\RequestBody(required: true, content: new OA\JsonContent(ref: '#/components/schemas/Epreuve')),
        responses: [
            new OA\Response(response: 200, description: 'Épreuve modifiée', content: new OA\JsonContent(
                properties: [new OA\Property(property: 'data', ref: '#/components/schemas/Epreuve')]
            )),
            new OA\Response(ref: '#/components/responses/RequeteInvalide', response: 400),
            new OA\Response(ref: '#/components/responses/NonAuthentifie', response: 401),
            new OA\Response(ref: '#/components/responses/NonTrouve', response: 404),
            new OA\Response(ref: '#/components/responses/DonneesInvalides', response: 422),
            new OA\Response(ref: '#/components/responses/ErreurServeur', response: 500),
        ]
    )]
    public static function modifier(string $id): void
    {
        self::executer(function () use ($id) {
            $data = RequeteHTTP::getData();
            if (empty($data)) {
                ReponseHTTP::sendError("Aucune donnée fournie pour la modification", 400);
            }
            $service = new CalendrierService();
            return $service->modifier($id, $data);
        });
    }

    /**
     * Point d'entrée pour supprimer une épreuve par son numéro FFA.
     */
    #[OA\Delete(
        path: '/{id}', operationId: 'supprimer', summary: 'Supprime une épreuve', tags: ['Calendrier'],
        security: [['bearerAuth' => []]],
        parameters: [new OA\Parameter(ref: '#/components/parameters/IdFfa')],
        responses: [
            new OA\Response(response: 200, description: 'Épreuve supprimée', content: new OA\JsonContent(
                properties: [new OA\Property(property: 'data', type: 'string', example: 'Épreuve supprimée')]
            )),
            new OA\Response(ref: '#/components/responses/NonAuthentifie', response: 401),
            new OA\Response(ref: '#/components/responses/NonTrouve', response: 404),
            new OA\Response(ref: '#/components/responses/ErreurServeur', response: 500),
        ]
    )]
    public static function supprimer(string $id): void
    {
        self::executer(function () use ($id) {
            $service = new CalendrierService();
            return $service->supprimer($id);
        });
    }
}
```

### Étape 3 : Générer le fichier de spécification JSON

Pour que l'interface graphique lise ces attributs, lancez la commande suivante dans votre terminal à la racine du projet :

Bash

```
vendor/bin/openapi src/ -o public/openapi.json
```

Cette commande analyse le dossier `src/` et produit le fichier de liaison `public/openapi.json` indispensable pour Swagger UI.

### Étape 4 : Créer le fichier des schémas globaux et métadonnées (`OpenApi.php`)

En plus d'annoter les contrôleurs, Swagger a besoin d'un fichier global (souvent placé dans `src/Documentation/OpenApi.php`) pour définir les informations générales de l'API (titre, version, sécurité Bearer), ainsi que les schémas de données réutilisés (comme l'objet `Epreuve`).

Il faut créer ce fichier dans le projet (L'IA intégrée au projet peut prendre  en charge la génération de ce fichier):


```php
<?php
declare(strict_types=1);

namespace Documentation;

use OpenApi\Attributes as OA;

#[OA\Info(
    version: '1.0.0',
    title: 'API Calendrier',
    description: "API CRUD sur le calendrier des épreuves. Les routes GET sont publiques, les autres nécessitent un jeton Bearer."
)]
#[OA\Server(url: '/', description: 'Serveur courant')]
#[OA\SecurityScheme(securityScheme: 'bearerAuth', type: 'http', scheme: 'bearer')]

#[OA\Parameter(
    parameter: 'IdFfa', name: 'id', in: 'path', required: true,
    description: "Numéro d'épreuve FFA (6 chiffres)",
    schema: new OA\Schema(type: 'string', pattern: '^[0-9]{6}$'), example: '325932'
)]

#[OA\Response(
    response: 'RequeteInvalide', description: 'Requête invalide (corps absent ou vide)',
    content: new OA\JsonContent(ref: '#/components/schemas/Erreur')
)]
#[OA\Response(
    response: 'NonAuthentifie', description: 'Jeton Bearer absent ou invalide',
    content: new OA\JsonContent(ref: '#/components/schemas/Erreur')
)]
#[OA\Response(
    response: 'NonTrouve', description: "L'épreuve n'existe pas",
    content: new OA\JsonContent(ref: '#/components/schemas/Erreur')
)]
#[OA\Response(
    response: 'Conflit', description: "Le numéro d'épreuve est déjà attribué",
    content: new OA\JsonContent(ref: '#/components/schemas/Erreur')
)]
#[OA\Response(
    response: 'DonneesInvalides', description: 'Données invalides',
    content: new OA\JsonContent(ref: '#/components/schemas/Erreur')
)]
#[OA\Response(
    response: 'ErreurServeur', description: 'Erreur technique',
    content: new OA\JsonContent(ref: '#/components/schemas/Erreur')
)]
class OpenApi
{
}

#[OA\Schema(
    schema: 'Epreuve',
    required: ['id', 'date', 'nom', 'ville'],
    properties: [
        new OA\Property(property: 'id', type: 'string', example: '325932'),
        new OA\Property(property: 'date', type: 'string', format: 'date', example: '2027-01-09'),
        new OA\Property(property: 'nom', type: 'string', example: 'Course Monsboubert'),
        new OA\Property(property: 'ville', type: 'string', example: 'MONSBOUBERT'),
        new OA\Property(property: 'distances', type: 'string', example: '10 km et 3 km'),
        new OA\Property(property: 'challenge', type: 'integer', example: 0),
        new OA\Property(property: 'label', type: 'string', nullable: true, example: 'Bronze'),
        new OA\Property(property: 'organisateur', type: 'string', nullable: true, example: 'FOURNIER CHRISTIAN'),
        new OA\Property(property: 'email', type: 'string', format: 'email', nullable: true, example: 'christian.fournier77@sfr.fr'),
    ]
)]
class EpreuveSchema
{
}

#[OA\Schema(
    schema: 'Erreur',
    properties: [
        new OA\Property(property: 'error', type: 'object', example: ['id' => "Ce numéro d'épreuve est déjà attribué"])
    ]
)]
class ErreurSchema
{
}
```

Il est tout à fait légitime de se demander à quoi servent ces classes qui sont **totalement vides (`{}`)** ! 
Elles n'ont aucune méthode, aucun code métier, aucune propriété.

Voici pourquoi elles existent et pourquoi elles sont indispensables :

### La contrainte de la syntaxe PHP (Les attributs ont besoin d'un "support")

En PHP 8, les attributs (les fameux `#[...]`) ne peuvent pas flotter "dans le vide" au milieu d'un fichier. PHP exige qu'un attribut soit **obligatoirement rattaché à un élément du code** (une classe, une méthode, une propriété ou un paramètre).

- Quand on écrit une route comme `#[OA\Get]`, on peut la rattacher directement à une méthode existante (comme `getAll()` dans notre contrôleur).
    
- Mais pour des éléments **globaux** (comme le titre de l'API, la version, la sécurité, ou un modèle de données comme `Epreuve` qui n'appartient à aucun contrôleur en particulier), il faut bien les rattacher à _quelque chose_.
    
###  Le rôle de "porte-manteau" (ou d'ancrage)

Ces classes vides (`class OpenApi`, `class EpreuveSchema`) ne servent donc **strictement à rien pour l'exécution de votre site web**.

Leur unique rôle est de servir de **porte-manteau** (ou de support d'ancrage) pour y accrocher les attributs OpenAPI :

- La classe `OpenApi` sert de support pour l'attribut global `#[OA\Info]`.
- La classe `EpreuveSchema` sert de support pour l'attribut `#[OA\Schema(schema: 'Epreuve')]`.
###  Comment l'outil réagit-il ?

Quand on lance la commande de génération (`vendor/bin/openapi src/ -o public/openapi.json`), le paquet `zircote` parcourt tous tes fichiers PHP.

- Il arrive sur le fichier `OpenApi.php`.
- Il voit la classe `OpenApi` (il se moque de son nom et du fait qu'elle soit vide).
- Il lit l'attribut `#[OA\Info]` qui est posé juste au-dessus, extrait les informations, et les intègre dans le fichier JSON final.

### Étape 5 : Mettre en place l'interface graphique (Swagger UI)

Pour afficher la documentation sous forme de page web interactive, il faut créer un dossier `public/docs/` et y ajouter un fichier **`index.html`** :


```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>API Calendrier - Documentation</title>
    <!-- Chargement du design de Swagger UI depuis un CDN -->
    <link rel="stylesheet" href="https://unpkg.com/swagger-ui-dist@5/swagger-ui.css">
</head>
<body>
    <div id="swagger-ui"></div>
    <!-- Chargement du moteur JavaScript de Swagger -->
    <script src="https://unpkg.com/swagger-ui-dist@5/swagger-ui-bundle.js"></script>
    <script>
        // On indique à Swagger UI où trouver le fichier de spécifications JSON généré
        SwaggerUIBundle({url: '../openapi.json', dom_id: '#swagger-ui'});
    </script>
</body>
</html>
```

### Étape 6 : Générer le fichier `openapi.json`

C'est l'étape de compilation. Pour transformer tes attributs PHP et ton fichier de configuration en un plan lisible par Swagger UI, ouvre ton terminal à la racine du projet et tape la commande :

Bash

```
vendor/bin/openapi src/ -o public/openapi.json
```

- **Ce qui se passe :** L'outil analyse l'intégralité du dossier `src/`, lit les attributs `#[OA\...]` dans les contrôleurs et tes schémas, et produit le fichier `public/openapi.json` attendu par le fichier `index.html`.
    

### Étape 7 : Consulter et tester dans le navigateur

1. Assure-toi que ton serveur local PHP est lancé.
    
2. Ouvre ton navigateur et rends-toi à l'adresse de ta documentation (par exemple : `http://apicalendrier/docs/`).
    
3. L'interface graphique s'affiche avec toutes les routes, les codes d'erreur et les formulaires de test (_Try it out_).
    

> **Règle d'or du développeur :** Chaque fois que tu modifies une route, un paramètre ou un modèle de données dans ton code PHP, n'oublie pas de relancer la commande de l'Étape 6 pour mettre à jour ton `openapi.json` !

## 4. Synthèse 

- **Démarche _Code-First_ :** Avec `zircote/swagger-php`, on rédige la documentation directement dans le code source PHP à l'aide d'attributs PHP 8 (`#[OA\Get]`, `#[OA\Response]`, etc.).
    
- **Automatisation :** Un script ou une tâche de build régénère le fichier JSON à chaque modification structurelle de l'API.
    
- **Séparation des préoccupations :** Le code métier reste propre et structuré, tandis que la documentation se déploie de manière autonome sur une page web dédiée pour les développeurs tiers.