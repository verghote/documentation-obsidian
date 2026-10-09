La classe `ClasseTechnique\Menu` permet de générer dynamiquement un menu de navigation HTML (arborescent et responsive) en se basant sur une structure définie dans un fichier de configuration JSON.

# Prise en main rapide

### 1. Structure des fichiers

Pour intégrer le menu, assurez-vous d'avoir l'organisation suivante :

```
├── config/
│   └── menu.json          # Déclaration du menu (structure & liens)
├── css/
│   ├── menu.css           # Feuille de style du menu
│   └── logo.png           # Logo de l'application (si utilisé dans le brand)
└── view/
    └── header.php         # Fichier de rendu
```

### 2. Intégration dans le code (`view/header.php`)

Il suffit d'instancier la classe avec le chemin du fichier JSON de configuration, puis d'appeler la méthode `afficher()` :

```php
<?php
declare(strict_types=1);

use ClasseTechnique\Menu;

// 1. Instanciation avec le chemin vers la configuration JSON
$menu = new Menu(__DIR__ . '/../config/menu.json');
?>

<!-- 2. Inclusion de la feuille de style -->
<link rel="stylesheet" href="/css/menu.css">

<!-- 3. Rendu du menu HTML -->
<?= $menu->afficher() ?>
```

##  Configuration du menu (`config/menu.json`)

Le comportement et le contenu du menu se pilotent intégralement via le fichier JSON.
### Exemple complet de fichier JSON :


```json
{
  "brand": {
    "label": "Consultation & recherche",
    "href": "/",
    "icon": "/css/logo.png"
  },
  "items": [
    {
      "label": "Consultation",
      "children": [
        {
          "label": "Catégorie",
          "href": "/categorie/liste"
        },
        {
          "label": "Coureur",
          "href": "/coureur/liste"
        }
      ]
    },
    {
      "label": "Rechercher",
      "children": [
        {
          "label": "Un coureur sur son numéro de licence",
          "href": "/coureur/getbylicence"
        },
        {
          "label": "Un coureur sur son nom",
          "href": "/coureur/getbyname"
        }
      ]
    }
  ]
}
```

# Structure et Propriétés du JSON

### 1. La section `brand` _(optionnelle)_

Définit l'élément principal et le logo situé en haut à gauche du menu.

| **Clé** | **Type** | **Obligatoire** | **Description**                                                                                   |
| ------- | -------- | --------------- | ------------------------------------------------------------------------------------------------- |
| `label` | `string` | **Oui**         | Texte affiché à côté ou à la place de l'icône.                                                    |
| `href`  | `string` | non             | Lien de redirection (ex: `/` ou `/accueil`). Si omis, l'élément est rendu sous forme de `<span>`. |
| `icon`  | `string` | non             | Chemin relatif vers l'image du logo (ex: `/css/logo.png`).                                        |

### 2. Le tableau `items` _(obligatoire)_

Contient la liste des éléments de navigation de premier niveau.

Chaque objet du tableau `items` accepte les clés suivantes :

|**Clé**|**Type**|**Obligatoire**|**Description**|
|---|---|---|---|
|`label`|`string`|**Oui**|Libellé affiché dans le menu. Si absent, l'élément est ignoré.|
|`href`|`string`|non|URL de destination.|
|`children`|`array`|non|Liste d'éléments enfants (sous-menu). Génère automatiquement une flèche `▾`.|

> **Note :** La récursivité est supportée. Vous pouvez imbriquer des éléments enfants (`children`) à l'intérieur d'un autre sous-menu.

# Fonctionnalités & Classes CSS auto-générées

La classe injecte automatiquement des classes CSS spécifiques sur la balise `<li>` pour faciliter la mise en forme et l'interactivité :

- **Mise en valeur de la page courante (`class="active"`) :**
    
    La classe détecte automatiquement l'URL de la page actuellement visitée (via `$_SERVER['REQUEST_URI']`).
    
    - Si l'URL de l'élément (`href`) correspond exactement à la page en cours, la classe `active` lui est attribuée.
        
    - Si un sous-menu contient la page active, le parent reçoit également la classe `active` afin de garder le sous-menu visuellement ouvert ou en surbrillance.
        
- **Sous-menus (`class="has-submenu"`) :**
    
    Appliquée automatiquement sur tout élément possédant des enfants (`children`).
    
- **Boutons vs Liens :**
    
    - Si un élément avec sous-menu **n'a pas** de champ `href`, la classe génère un `<button type="button">` pour garantir l'accessibilité.
        
    - S'il possède un `href`, il est rendu sous forme de lien `<a>`.
        

## ⚠️ Gestion des Erreurs & Exceptions

Si le fichier JSON n'est pas valide ou introuvable, la classe lève une exception de type **`ClasseTechnique\UserException`** :

|**Cas d'erreur**|**Message / Cause**|
|---|---|
|**Fichier inexistant**|`Fichier de menu introuvable : {chemin}`|
|**Échec de lecture**|`Impossible de lire le fichier : {chemin}`|
|**Erreur de syntaxe JSON**|`Erreur JSON : {message_erreur_json}`|
|**Format incorrect**|`La configuration du menu doit être un tableau JSON.`|