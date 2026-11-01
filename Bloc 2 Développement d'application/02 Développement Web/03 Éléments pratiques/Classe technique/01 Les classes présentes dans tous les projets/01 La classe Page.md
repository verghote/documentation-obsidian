# Présentation

`Page` représente une page HTML en cours de construction.

Elle est utilisée par le contrôleur pour décrire tout ce qui est nécessaire à la génération de la réponse :

- le titre de la page ;
- les feuilles de style CSS à charger ;
- les fichiers JavaScript à charger ;
- les données mises à disposition du JavaScript ;
- l'activation éventuelle de la protection CSRF.

La classe **ne génère pas directement le HTML**.

Une fois configurée, elle est transmise au template `interface.php`, qui se charge de produire le document HTML final.

# Principe d'utilisation

Une page est généralement créée par le contrôleur, configurée puis affichée.

Exemple :

```php
$page = new Page();  

$page->setTitre("Liste des catégories")  
    ->setDonnee("lesCategories", Categorie::getAll())  
    ->addScript("/composant/html2pdf/html2pdf.bundle.min.js")  
    ->afficher();
```

L'API est fluide : la plupart des méthodes retournent l'objet lui-même afin de pouvoir être chaînées.

# Définir le titre

Le titre de la page est défini avec :

```php
$page->setTitre('Liste des catégories');
```

Le titre peut être récupéré avec : $titre = $page->getTitre();

Le template 'view/interface.php' qui construit toutes les pages du site utilise ce titre dans la balise `<title>` 

```php
<title><?= $page->getTitre() ?></title>
```

# Ajouter des feuilles de style

Une feuille de style est ajoutée avec :

```php
$page->addStyle('/css/site.css');
```

Les doublons sont automatiquement ignorés.

Il est donc possible d'appeler plusieurs fois la méthode sans risque.

La liste des feuilles de style est disponible avec :

```php
$styles = $page->getStyles();
```

# Ajouter des scripts JavaScript

Un script est ajouté avec :

```php
$page->addScript('/js/application.js');
```

Comme pour les feuilles de style, un même fichier n'est ajouté qu'une seule fois.

La liste des scripts est accessible via :

```php
$scripts = $page->getScripts();
```

# Ajouter un composant

Lorsqu'un composant respecte la convention du framework :

```
/composant/
    nom/
        nom.min.js
        nom.css
```

il peut être ajouté simplement :

```php
$page->addComposant('datatable');
```

Cette instruction ajoute automatiquement :

```
/composant/datatable/datatable.min.js
/composant/datatable/datatable.css
```

Le nom du composant ne peut contenir que :

- des lettres ;
- des chiffres ;
- `_`
- `-`

Un nom invalide provoque une exception.

# Transmettre des données au JavaScript

Le contrôleur peut mettre des données à disposition du code JavaScript de la page.

Exemple :

```php
$page->setDonnee('utilisateur', $utilisateur);
$page->setDonnee('configuration', $configuration);
```

Ces données seront injectées par le template view/interface.php dans un bloc JSON.

Toutes les données sont accessibles côté serveur avec :

```php
$donnees = $page->getDonnees();
```

Coté client la fonction getData() de la bibliothèque de fonction public/composant/fonction/data.js

```javascript
import {getData} from "/composant/fonction/page.js";  
 
const utilisateur = getData('utilisateur');
```
# Validation de l'origine des appels par un jeton de session ou jeton CSRF

Toutes les pages n'ont pas besoin d'un jeton CSRF.

Une page contenant uniquement de la consultation n'en nécessite généralement pas.

En revanche, lorsqu'une page permet :

- une création ;
- une modification ;
- une suppression ;
- un appel AJAX protégé ;

le contrôleur active la génération du jeton :

```php
$page->avecJeton();
```

Le template view/interface.php détecte ensuite cette demande grâce à :

```php
$page->necessiteUnJeton();
```

et peut générer le jeton CSRF avant l'envoi de la page.

```php
<!DOCTYPE HTML>  
<html lang="fr">  
<head>  
    <title>Le téléversement</title>  
    <meta charset="utf-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1">  
    <!-- Création et intégration du token CSRF s'il est demandé -->  
    <?php if ($page->necessiteUnJeton()) : ?>  
        <meta name="csrf-token" content="<?= Jeton::creer() ?>">  
    <?php endif; ?>
```

La classe `Page` ne crée jamais le jeton elle-même.

En plaçant le jeton dans une balise `<meta>`, il devient disponible pour tous les scripts JavaScript de la page et notamment pour la fonction appelAjax de la bibliothèque fonction.ajax.js qui va automatiquement transmettre le jeton dans l'entête X-CSRF-Token

```javascript
const _csrfToken = document.querySelector('meta[name="csrf-token"]')?.content ?? null;

export async function appelAjax({url, data = null, method = 'POST', success = null, error = null, dataType = 'json'}) {
   ...
   const headers = {  
    'Accept': 'application/json',  
    'X-Requested-With': 'XMLHttpRequest',  
   };  
  
   // Ajouter le token CSRF si disponible (GET et POST)  
   if (_csrfToken) {  
       headers['X-CSRF-Token'] = _csrfToken;  
}
```

### Schéma de fonctionnement du Jeton CSRF

```
┌─────────────────────────────────────────────────────────────────────────┐
│ 1. GÉNÉRATION ET INJECTION (Serveur PHP -> Page HTML)                   │
└─────────────────────────────────────────────────────────────────────────┘
   [ Contrôleur ]  ──( $page->avecJeton() )──>  [ Vue PHP / Template ]
                                                       │
                                            ( Jeton::creer() )
                                                       │
                                                       ▼
   Génération du HTML envoyé au navigateur :
   <meta name="csrf-token" content="abc123xyz...">

───────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────┐
│ 2. LECTURE ET TRANSMISSION (Client JS -> Requête AJAX)                  │
└─────────────────────────────────────────────────────────────────────────┘
   [ Document HTML ]
          │
  ( document.querySelector )
          │
          ▼
   [ fonction.ajax.js ] ──( Ajout de l'en-tête )──> En-têtes HTTP :
                                                    Accept: application/json
                                                    X-CSRF-Token: abc123xyz...

───────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────┐
│ 3. VÉRIFICATION DE SÉCURITÉ (Serveur PHP / API)                         │
└─────────────────────────────────────────────────────────────────────────┘
   [ Script PHP / Action ] 
          │
          ├─► 1. Lit l'entête : $_SERVER['HTTP_X_CSRF_TOKEN']
          ├─► 2. Compare avec le jeton stocké en SESSION
          │
          ├── [ Valide ]   ──► Exécute l'action (Création/Modif/Suppr)
          └── [ Invalide ] ──► Rejette la requête (Erreur 403 Forbidden)
```

Dans notre architecture, le jeton ne sert pas à identifier un utilisateur connecté, mais à **garantir qu'un script d'action ne peut être exécuté que s'il provient d'une page légitimement chargée sur notre site**.

Cette mécanique empêche qu'un tiers ou un script externe puisse exécuter vos traitements PHP directement en contournant l'interface de l'application.

Dans ce cas on emploi plus communément le terme de **Jeton de Session**
# Afficher la page

Lorsque toutes les informations ont été définies, la page est générée avec :

```php
$page->afficher();
```

Cette méthode :

1. vérifie la présence du template `view/interface.php` ;
2. transmet l'objet `Page` au template ;
3. génère le document HTML ;
4. termine immédiatement l'exécution du script.

La méthode ne retourne jamais.

# Cycle de fonctionnement

Le cycle habituel est le suivant :

```
Contrôleur
      ▼
Création de la page
      ▼
Définition du titre
      ▼
Ajout des styles
      ▼
Ajout des scripts
      ▼
Ajout des données JavaScript
      ▼
Activation éventuelle du jeton CSRF
      ▼
Affichage de la page
      ▼
Template interface.php
      ▼
HTML envoyé au navigateur
```

# Responsabilités

La classe `Page` ne fait que décrire une page.

Elle ne réalise pas :

- la génération du HTML ;
- la création du jeton CSRF ;
- l'envoi des en-têtes HTTP ;
- le rendu des balises `<script>` ou `<link>`.

Ces responsabilités appartiennent au template view/interface.php.

Cette séparation permet au contrôleur de décrire la page sans se préoccuper de sa représentation HTML.

# Exemple complet

```php
declare(strict_types=1);  
  
use ClasseMetier\Club;  
use ClasseTechnique\Page;  
  
// activation du chargement dynamique des ressources  
require $_SERVER['DOCUMENT_ROOT'] . "/../bootstrap/autoload.php";  
  
// récupération des Clubs  
$lesClubs = Club::getAll();  
  
// alimentation et affichage de l'interface  
$page = new Page();  
$page->setDonnee('lesClubs', $lesClubs)  
    ->avecJeton()  
    ->afficher();
```

Le template pourra alors :

- utiliser le titre ;
- générer les balises `<link>` ;
- générer les balises `<script>` ;
- transmettre les données au JavaScript ;
- créer un jeton CSRF si nécessaire ;
- produire le document HTML complet.

# Résumé

`Page` est l'objet utilisé par le contrôleur pour préparer une réponse HTML.

Elle centralise toutes les informations nécessaires à la construction d'une page :

- titre ;
- feuilles de style ;
- scripts JavaScript ;
- composants ;
- données destinées au JavaScript ;
- nécessité d'un jeton CSRF.

Une fois configurée, elle est simplement transmise au template chargé de produire le document HTML final.