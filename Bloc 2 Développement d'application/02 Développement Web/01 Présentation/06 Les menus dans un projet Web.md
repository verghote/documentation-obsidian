  
Le projet met en place trois types de menus ayant chacun un rôle précis :  
  
1. le menu vertical global ;  
2. le menu horizontal de module ;  
3. le menu placé dans le header.  
  
`InterfaceHtml` montre clairement cette organisation : le header est chargé comme fragment commun, le menu vertical est injecté globalement et le menu horizontal est injecté au niveau du module (`src/ClasseTechnique/InterfaceHtml.php`).  
  
## 1. Le menu vertical global  
  
Le menu vertical est le moins utilisé dans les applications web classiques, mais il reste très utile lorsqu'un projet propose :  
  
- beaucoup d'actions permanentes ;  
- une navigation de type application métier ;  
- un accès rapide aux grandes rubriques du site.  
  
Dans un projet, il se présente comme un menu global affiché sur le côté gauche.  
  
### Emplacement de la configuration  
  
Le menu vertical est recherché automatiquement dans :  
  
```text  
/config/menuvertical.json  
```  
  
Le chargement est effectué par :  
  
```php  
$fichier = DOSSIER_RACINE . '/config/menuvertical.json';  
```  
  
Si ce fichier n'existe pas, aucun menu vertical n'est affiché.  
  
### Fonctionnement technique  
  
Le menu vertical est injecté par `InterfaceHtml::menuVertical()`, qui ajoute un script module :  
  
```javascript  
import { initialiserMenuVertical } from "/composant/menuvertical/menu.js";  
initialiserMenuVertical($json, 150);  
```  
  
Le composant JavaScript :  
  
- injecte automatiquement la feuille de style ;  
- crée une barre latérale `<nav id="sidebar">` ;  
- génère les liens à partir du JSON ;  
- gère le repli du menu ;  
- mémorise l'état replié dans `localStorage` ;  
- adapte l'affichage en version mobile.  
  
Le composant utilisé se trouve dans :  
  
```text  
/public/composant/menuvertical/menu.js  
/public/composant/menuvertical/menu.css  
```  
  
### Structure attendue  
  
Le fichier JSON doit contenir une liste d'options :  
  
```json  
[  
  {    "label": "Accueil",    "href": "/"  },  
  {    "label": "Clubs",    "href": "/club/liste"  }
]  
```  
  
Chaque entrée fournit :  
  
- `label` : le texte affiché ;  
- `href` : l'URL de destination.  
  
### Usage recommandé  
  
Ce menu convient surtout pour :  
  
- des applications d'administration ;  
- des back-offices ;  
- des outils métier avec navigation persistante.  
  
Il est moins fréquent sur les sites vitrines ou sur les interfaces très simples.  
  
## 2. Le menu horizontal de module  
  
Le menu horizontal permet de parcourir facilement les interfaces d'un module. 
C'est son intérêt principal : proposer, au même endroit, les actions usuelles d'une rubrique fonctionnelle.  
  
Dans ce projet, il est prévu pour les sous-applications situées dans `public`.  
  
### Emplacement de la configuration  
  
Le fichier est recherché automatiquement dans le sous-répertoire `config` du module courant :  
  
```text  
/public/<module>/config/menuhorizontal.json  
```  
  
Le chargement est réalisé par :  
  
```php  
$fichier = $this->repertoirePage . '/../config/menuhorizontal.json';  
```  
  
Exemples présents dans le projet :  
  
```text  
/public/document/config/menuhorizontal.json  
/public/club/config/menuhorizontal.json  
```  
  
### Fonctionnement technique  
  
`InterfaceHtml::menuHorizontal()` injecte un script module :  
  
```javascript  
import { initialiserMenuHorizontal } from "/composant/menuhorizontal/menu.js";  
initialiserMenuHorizontal($json);  
```  
  
Le composant JavaScript :  
  
- ajoute la feuille de style du menu horizontal ;  
- crée un `<nav id="menuHorizontal">` ;  
- insère le menu au début du `<main>` ;  
- marque automatiquement l'option active selon l'URL courante.  
  
Le composant utilisé se trouve dans :  
  
```text  
/public/composant/menuhorizontal/menu.js  
```  
  
### Structure attendue  
  
Le fichier JSON contient une liste d'actions du module. Exemple pour le module `document` :  
  
```json  
[  
	{    
		"label": "👁️",  
	    "href": "/document/liste",    
	    "title": "Liste des documents"  
	},  
	{    
		"label": "➕",  
	    "href": "/document/ajout",
	    "title": "Ajouter un nouveau document"
	},  
	{    
		"label": "✏️",  
	    "href": "/document/maj",    
	    "title": "Mettre à jour un document"  
	}
]  
```  
  
Chaque entrée peut contenir :  
  
- `label` : texte ou pictogramme affiché ;  
- `href` : URL de destination ;  
- `title` : infobulle décrivant l'action.  
  
### Usage recommandé  
  
Ce menu est particulièrement adapté pour :  
  
- lister les pages d'un même module ;  
- accéder rapidement aux écrans liste, ajout, mise à jour ;  
- garder une navigation compacte et proche du contenu métier.  
  
C'est le menu le plus pratique pour naviguer dans les différentes interfaces d'un module.  
  
## 3. Le menu placé dans le header  
  
Le menu du header correspond à la navigation principale de l'application. Il est visible en haut de page et sert à accéder aux grandes rubriques du site.  
  
Dans ce projet, il est généré par la classe `ClasseTechnique\Menu`.  
  
### Emplacement de la configuration  
  
Le header charge le fichier :  
  
```text  
/config/menu.json  
```  
  
Depuis `view/header.php` :  
  
```php  
$menu = new Menu(__DIR__ . '/../config/menu.json');  
```  
  
### Fonctionnement technique  
  
Le fragment commun `view/header.php` :  
  
- instancie la classe `Menu` ;  
- charge la feuille de style `/css/menu.css` ;  
- affiche le HTML généré par `$menu->afficher()`.  
  
La classe `Menu` prend en charge :  
  
- le chargement du JSON ;  
- la marque de l'élément actif selon l'URL ;  
- la gestion d'un brand ;  
- la génération de sous-menus éventuels ;  
- l'échappement HTML des valeurs affichées.  
  
### Structure attendue  
  
Le fichier `config/menu.json` utilise un objet avec deux parties :  
  
- `brand` ;  
- `items`.  
  
Le `brand` permet d'afficher :  
  
- un libellé ;  
- un lien de retour ;  
- éventuellement une icône.  
  
Les `items` définissent les entrées principales de navigation.  
  
### Usage recommandé  
  
Le menu du header convient pour :  
  
- la navigation principale du site ;  
- l'accès aux grands modules ;  
- la mise en avant de l'identité de l'application grâce au brand.  
  
Il s'agit du menu le plus visible et le plus structurant pour l'utilisateur.  

Exemple d'un menu sans sous menu :  
  
```json  
{  
  "brand": {  
    "label": "Téléversement",  
    "href": "/",  
    "icon": "/css/logo.png"  
  },  
  "items": [  
    {  
      "label": "Classement",  
      "href": "/uploadclassement/"  
    },  
    {  
      "label": "Photo",  
      "href": "/uploadimage/"  
    },  
    {  
      "label": "Document",  
      "href": "/document/liste"  
    },  
    {  
      "label": "Club",  
      "href": "/club/liste"  
    }  
  ]  
}
```  

Les items de ce type de menu peuvent facilement être complété avec un menu horizontal
C'est de loin la meilleure solution au niveau d'un smartphone

Exemple de menu avec des sous menu

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
        },  
        {  
          "label": "Club",  
          "href": "/club/liste"  
        },  
        {  
          "label": "Annonce",  
          "href": "/annonce/liste"  
        },  
        {  
          "label": "Projet",  
          "href": "/projet/liste"  
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
          "label": "Un coureur sur son nom avec autocomplétion",  
          "href": "/coureur/getbyname"  
        },  
        {  
          "label": "Les coureurs dans une catégorie",  
          "href": "/coureur/getbycategorie"  
        },  
        {  
          "label": "Recherche par catégorie, sexe et club",  
          "href": "/coureur/recherchemultiple"  
        },  
        {  
          "label": "Des compétences par bloc et par domaine",  
          "href": "/competence"  
        }  
      ]  
    },  
  
    {  
      "label": "Filtrer les coureurs",  
      "children": [  
        {  
          "label": "Par catégorie, sexe et club",  
          "href": "/coureur/filtragemultiple"  
        },  
        {  
          "label": "Sur plusieurs colonnes",  
          "href": "/coureur/filtrer"  
        }  
      ]  
    }  
  ]  
}
```
## Synthèse  
  
| Type de menu | Emplacement | Configuration | Rôle principal |  
|---|---|---|---|  
| Menu vertical | côté gauche | `/config/menuvertical.json` | navigation globale persistante |  
| Menu horizontal | dans le contenu du module | `/public/<module>/config/menuhorizontal.json` | navigation entre les interfaces d'un module |  
| Menu du header | en haut de page | `/config/menu.json` | navigation principale de l'application |  
  
Le choix du menu dépend donc du niveau de navigation visé :  
  
- **header** pour les grandes rubriques ;  
- **horizontal** pour les actions d'un module ;  
- **vertical** pour une navigation métier plus dense et permanente.