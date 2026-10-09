## 1. Rôle des scripts 

Les scripts PHP placés dans le sous-répertoire ajax constituent les **points d'entrée serveur utilisés par les traitements Ajax du module**.

Ils sont appelés par la fonction JavaScript commune :  **appelAjax()**

L'appel est réalisé de manière asynchrone, sans recharger la page.

Le principe général est :

```text
Page HTML
    │
    ▼
JavaScript
    │
    ▼
appelAjax()
    │
    │ requête HTTP
    ▼
ajax/script.php
    │
    ├── contrôle de la requête utilisée pour transmettre les paramètres : POST ou GET : Requete::exigerPost() ou RequeteExigerGet() principalement
    ├── récupération et vérification du type des paramètres : Requete::getString(), Requete::postString() ou  autres méthodes typées
    ├── validation des données : test par expression régulière le plus souvent
    ├── contrôle en base de données : appel des méthodes de la classe métier (getByid, exists, etc.)
    └── traitement métier et génération de la réponse au format JSON
    │
    ▼
ReponseJson
    │
    └── réponse JSON : méthodes de la classe ReponseJson 
    │
    ▼
appelAjax() : traitement automatique de la réponse JSON correspondant à une erreur
    │
    ▼
Mise à jour de l'interface : fonction de rappel 'success' associé à la fonction appelAjax
``` 

# 2. Organisation des scripts Ajax

Tous les scripts PHP appelés par Ajax sont placés dans un sous-répertoire `ajax` du module.

Exemple :

```text
module/
├── index.php
├── coureur.php
├── coureur.js
│
└── ajax/
    ├── getlescompetences.php
    ├── getcoureur.php
    └── getclub.php
```

Cette organisation permet d'identifier immédiatement les scripts qui constituent des points d'entrée Ajax.

# 3. Appel par `appelAjax()`

Les scripts Ajax ne construisent pas directement une page HTML.

Ils reçoivent une requête HTTP et retournent une réponse destinée au JavaScript.

L'appel est effectué par :

```javascript
appelAjax()
```

Le principe est généralement :

```text
JavaScript
     │
     │ GET / POST
     │
     ▼
ajax/getlescompetences.php
     │
     │ JSON
     ▼
JavaScript
```

La page affichée dans le navigateur n'est donc pas rechargée.

AJAX signifie :

```text
Asynchronous JavaScript And XML
```

Même si le terme historique fait référence à XML, l'application utilise ici principalement **JSON** pour échanger les données entre JavaScript et PHP.

L'intérêt est de pouvoir demander ou modifier des données sans recharger la page complète.

Par exemple :

```text
Utilisateur sélectionne un projet
          │
          ▼
appelAjax()
          │
          ▼
getlescompetences.php
          │
          ▼
Recherche en base
          │
          ▼
Réponse JSON
          │
          ▼
JavaScript
          │
          ▼
Mise à jour des compétences
```

# Les protections

Une première protection définie dans le fichier public/.htaccess est mis en place au niveau du serveur Apache 

Le fichier `.htaccess` contient :

```apache
RewriteCond %{REQUEST_URI} /ajax/
RewriteCond %{HTTP:X-Requested-With} !^XMLHttpRequest$ [NC]
RewriteRule ^ - [F,L]
```

La première condition vérifie que l'URL demandée contient /ajax/

La seconde vérifie test si la requête ne possède pas l'en-tête : X-Requested-With: XMLHttpRequest

Si c'est le cas l'action suivante est réalisée :  RewriteRule ^ - [F,L]

Apache refuse la requête avec une erreur 403 Forbidden


L'en-tête :  X-Requested-With: XMLHttpRequest permet d'identifier les requêtes Ajax classiques.

La règle Apache permet donc notamment d'empêcher un accès accidentel direct du type :  https://monsite/module/ajax/getcoureur.php dans un navigateur.

Cependant, cet en-tête **ne constitue pas à lui seul une protection de sécurité suffisante** : un client HTTP peut techniquement fabriquer cet en-tête.

La protection Apache doit donc être considérée comme une **première barrière**, et non comme une preuve qu'une requête provient réellement de `appelAjax()`.


Une seconde protection est mise en place au niveau de l'application avec un jeton.

Le principe est de transmettre un jeton avec la requête Ajax.

Le serveur peut alors vérifier que la requête possède bien le jeton attendu.

Le mécanisme général est :

```text
Page
 │
 ├── reçoit/crée le jeton
 │
 ▼
JavaScript
 │
 │ jeton
 ▼
appelAjax()
 │
 ▼
Script Ajax
 │
 └── vérification du jeton
```

Cette protection complète la restriction Apache.

Le principe est le suivant :

- Le serveur génère un identifiant aléatoire unique appelé jeton CSRF.
- Ce jeton est conservé côté serveur dans une variable de session.
- Il est également transmis à la page Web dans une balise <meta> (ou éventuellement dans un champ caché d'un formulaire, ou dans un cookie).
- La page Web réalise un appel Ajax en transmettant la valeur du token récupérée dans la balise meta, dans l'entête HTTP : X-CSRF-Token
- Le serveur compare alors la valeur reçue avec celle qu'il a conservée en session.
- Il vérifie également que le jeton n'a pas expiré.
- Si le jeton est absent, invalide ou expiré, la requête est rejetée.

  En AJAX pur, la balise <meta> est souvent préférable car il évite la dépendance aux formulaires HTML.

Le jeton CSRF est propre à la session utilisateur et change périodiquement afin d’empêcher toute réutilisation depuis une autre session.

Cette technique permet de s'assurer que la requête provient bien d'une page générée par l'application et non d'une requête forgée par un tiers.

La classe Jeton implémente l'ensemble de ce mécanisme : génération du jeton, stockage en session, transmission au client et vérification lors des appels AJAX.

La mise en place dans une application repose sur les étapes suivantes :

- Le script index.php ayant besoin d'activer une protection par jeton demande la création d'un jeton en utilisant  la méthode jeton de la classe Page

```php
$page = new Page();

$page->avecJeton()
	 ->afficher();
```

Il peut passer en paramètre la durée de validité du jeton en seconde 

```php
$page->Jeton(300)
```

Le jeton a une durée de validité de 5 minutes, en absence de valeur le jeton est valable pendant toute la session

- Le script index.js associé au script index.php réalise l'appel ajax via la fonction appelAjax

```javascript
appelAjax({
        url: '/ajax/x.php',
        data: {cle : valeur, …},
        success: data => { …};
});
```

*En utilisant cette fonction, le passage du token récupéré dans la balise meta est transmis automatiquement dans l'entête de la requête

- Le script x.php appelé dans l'exemple doit juste appeler la méthode Requete::exigerPost() pour vérifier la présence du jeton, et sa validité

# 4. Modèle général d'un script PHP appelé  en Ajax

Un script Ajax peut généralement suivre cette structure :

```php
<?php

declare(strict_types=1);

use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;

require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/bootstrap.php';

// 1. Vérification de la méthode HTTP
Requete::exigerGet();

// 2. Récupération des paramètres
$parametre = Requete::getString('parametre');

// 3. Validation
if (trim($parametre) === '') {

    ReponseJson::envoyerLesErreurs(['global' => 'Paramètre obligatoire.']);
}

// 4. Contrôle du format
// ...

// 5. Appel de la classe métier
$resultat = /* classe métier */;

// 6. Réponse
ReponseJson::envoyerLesDonnees($resultat);
```

Ce modèle constitue une **trame générale**. Les contrôles et les méthodes `Requete` doivent être adaptés aux besoins de chaque script.

# 5. Contrôle de la méthode HTTP

Chaque script Ajax commence par vérifier que la méthode HTTP utilisée correspond à celle attendue.

par exemple Requete::exigerGet()  exige donc une requête GET

Si le script doit recevoir des données par POST, il faut utiliser  Requete::exigerPost() 

**La méthode exigerPost() vérifie aussi la présence et la validité du jeton.**

Si l'on en souhaite pas vérifier le jeton on doit utiliser la méthode **exigerPostSansJeton()**

Le contrôle de la méthode permet de définir clairement le contrat du script.

GET → récupérer des données
POST → transmettre des données ou effectuer une opération

# 6. Récupération des paramètres avec `Requete`

Dans l'exemple :

```php
$search = Requete::getString('search');
```

La valeur du paramètre search est récupérée par la classe  Requete

La classe Requete centralise les opérations de récupération et de contrôle des paramètres HTTP à l'aide des méthodes get() ou  post() ou plus précisément par ses méthodes typées. `getString()`, `getInt()`, `postString()`, `postInt()`, etc.

Sans classe spécialisée, le développeur devrait utiliser les mécanismes PHP natifs :

```php
// Vérification de la méthode HTTP
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    ReponseJson::envoyerLesErreurs(['global' => 'La méthode HTTP POST est requise.']);
}

// Vérification de la présence du paramètre
if (!isset($_POST['idProjet'])) {
    ReponseJson::envoyerLesErreurs(['idProjet' => "Le paramètre 'idProjet' est obligatoire."]);
}

// Vérification que la valeur est un entier
$idProjet = filter_var($_POST['idProjet'], FILTER_VALIDATE_INT);

if ($idProjet === false) {
    ReponseJson::envoyerLesErreurs(['idProjet' => "Le paramètre 'idProjet' doit être un nombre entier."]);
}

```

ou :

```php
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    ReponseJson::envoyerLesErreurs(['global' => 'La méthode POST est requise.']);
}

$idProjet = filter_input(INPUT_POST, 'idProjet', FILTER_VALIDATE_INT);

if ($idProjet === null) {
    ReponseJson::envoyerLesErreurs(['idProjet' => "Le paramètre 'idProjet' est obligatoire."]);
}

if ($idProjet === false) {
    ReponseJson::envoyerLesErreurs(['idProjet' => "Le paramètre 'idProjet' doit être un entier."]);
}

```

PHP fournit différents filtres, notamment :

```text
FILTER_VALIDATE_INT
FILTER_VALIDATE_FLOAT
FILTER_VALIDATE_EMAIL
FILTER_VALIDATE_URL
FILTER_VALIDATE_IP
FILTER_VALIDATE_REGEXP
```

Cependant, répéter ces contrôles dans chaque contrôleur conduirait à multiplier le code.

La classe `Requete` permet de centraliser ces opérations.

# 7. Contrôle du contenu des paramètres

La vérification de la méthode HTTP ne suffit pas.

Un paramètre correctement transmis peut néanmoins contenir une valeur incorrecte.

Le script doit donc vérifier :

```text
présence
   │
   ▼
type
   │
   ▼
format
   │
   ▼
valeur autorisée
   │
   ▼
existence en base
   │
   ▼
droits éventuels
```

Cette succession de contrôles permet de ne pas faire confiance aux données reçues du navigateur.

Exemple : tester une chaîne vide

```php
if (trim($search) === '') {
    ReponseJson::envoyerLesErreurs(['nomR' => "Le paramètre 'search' est vide."]);
}
```

Une chaîne vide est donc refusée.

Exemple : tester le format

```php
if (!preg_match("/^[\p{L} '-]+$/u", $search)) {
    ReponseJson::envoyerLesErreurs(['nomR' => "Seules les lettres et les espaces sont autorisées"]);
}
```

L'expression régulière permet d'autoriser notamment :

- les lettres ;
- les espaces ;
- l'apostrophe ;
- le trait d'union.

Le modificateur /u permet de traiter correctement les caractères Unicode.

C'est notamment nécessaire pour les caractères accentués.

Exemple : teste de l'existence  au niveau de la base de données

```php
$coureur = Coureur::getByLicence($licence);  
if (!$coureur) {  
    ReponseJson::envoyerLesErreurs(['licence' => 'Numéro de licence inexistant.']);  
}
```

# 8. Envoi de la réponse avec `ReponseJson`

Les scripts Ajax envoient toujours leur réponse dans le format JSON à l'aide des méthode de la classe ReponseJson

Deux situations principales sont prévues.

Réponse positive 

```php
ReponseJson::envoyerLesDonnees($donnees);
```
### Réponse d'erreur

```php
ReponseJson::envoyerLesErreurs(['nomR' => "Le paramètre 'search' est vide."]);
```

La méthode envoyerLesErreurs() arrête le traitement du script et retourne une réponse JSON.

la clé  'nomR' permet d'indiquer à quel élément l'erreur est associée.

Cette information est exploitée par le traitement JavaScript commun.

Elle permet de distinguer notamment une erreur globale (clé 'global') d'une erreur liée à un champ

La fonction appelAjax prend en charge le traitement de ces erreurs.

Une erreur 'global' sera affichée dans la balise  :

```html
<div id="msg">
```

si cet élément existe ;  sinon dans une fenêtre modale.

Si la clé correspond à un champ du formulaire, la fonction appelAjax affichera l'erreur en dessous du champ du même 'id'.
Si ce champ n'est pas trouvé, l'erreur est affiché dans une boîte de dialogue.

# 9. Validation et sécurité

La validation des paramètres reçus par Ajax est indispensable.

Il ne faut jamais considérer qu'une donnée est fiable simplement parce qu'elle provient de l'application JavaScript.

Le navigateur peut envoyer une requête différente de celle prévue par l'interface.

Le serveur doit donc toujours vérifier les données.

Le principe est :

> **Le JavaScript facilite l'utilisation de l'application, mais le serveur reste responsable de la validation et de la sécurité.**

# 10. Résumé

Les scripts PHP du répertoire `ajax` constituent les points d'entrée serveur utilisés par `appelAjax()`.

Ils suivent un fonctionnement commun :

```text
requête Ajax
     │
     ▼
protection Apache
     │
     ▼
contrôleur Ajax
     │
     ├── méthode HTTP
     ├── jeton / sécurité
     ├── récupération des paramètres
     ├── validation
     ├── contrôles fonctionnels
     │
     ▼
classe métier
     │
     ▼
ReponseJson
     │
     ▼
réponse JSON
     │
     ▼
appelAjax()
```

L'utilisation systématique de `Requete` et `ReponseJson` permet d'obtenir un fonctionnement homogène de l'ensemble des contrôleurs Ajax.

Le principe essentiel est :

> **Le contrôleur Ajax reçoit, contrôle et transmet ; la classe métier traite ; `ReponseJson` répond ; `appelAjax()` exploite la réponse.**