## 1. Vue d'ensemble

Les fichiers `interface.php` et `InterfaceHtml.php` travaillent ensemble pour construire l'interface HTML générale des pages de l'application.

Le principe est le suivant :

```text
Page applicative
      │
      ▼
interface.php
      │
      ▼
InterfaceHtml
      │
      ├── <head>
      │    ├── CSRF
      │    ├── Bootstrap
      │    ├── menus
      │    ├── CSS
      │    ├── JavaScript
      │    └── données JavaScript
      │
      ├── header
      │
      ├── contenu de la page
      │
      └── footer
```

`interface.php` définit la **structure HTML générale**.

`InterfaceHtml.php` prend en charge la **construction dynamique des différents éléments de cette structure**.

# 2. Rôle de `interface.php`

Le fichier `interface.php` constitue le **gabarit HTML commun des pages**.

Il contient la structure générale :

```html
<!DOCTYPE html>
<html>
<head>
    ...
</head>
<body>
    ...
</body>
</html>
```

Il ne construit pas directement les différents éléments de l'interface.

Il délègue cette responsabilité à la classe :

```php
ClasseTechnique\InterfaceHtml
```

# 3. Déclaration du typage strict

Le fichier commence par :

```php
declare(strict_types=1);
```

Le typage strict est donc activé pour `interface.php`.

---

# 4. Annotation de la variable `$page`

Le fichier contient :

```php
/** @var \ClasseTechnique\Page $page */
```

Cette annotation indique que la variable `$page` est supposée être une instance de :

```php
ClasseTechnique\Page
```

Cette information est principalement destinée aux outils de développement et à l'analyse statique.

Elle permet notamment à un IDE de connaître le type de `$page` et de proposer les méthodes disponibles.

---

# 5. Importation de `InterfaceHtml`

Le fichier utilise :

```php
use ClasseTechnique\InterfaceHtml;
```

La classe peut ensuite être instanciée simplement avec :

```php
$html = new InterfaceHtml($page);
```

---

# 6. Création de l'interface

L'objet `InterfaceHtml` reçoit la page courante :

```php
$html = new InterfaceHtml($page);
```

La classe conserve cette page dans sa propriété :

```php
private Page $page;
```

Elle peut ainsi récupérer les informations déclarées dans `Page`, notamment :

- le titre ;
- les feuilles de style ;
- les scripts ;
- les données JavaScript ;
- l'indication concernant le token CSRF.

---

# 7. Structure générale de `interface.php`

Le fichier produit une structure HTML classique :

```php
<!DOCTYPE html>
<html lang="fr">
<head>
    ...
</head>
<body>

    ...

    <main>
        ...
    </main>

    ...

</body>
</html>
```

Les différents éléments sont délégués à `InterfaceHtml`.

---

# 8. Élément `<title>`

Le titre de la page est obtenu avec :

```php
$page->getTitre()
```

Le code :

```php
<title><?= $page->getTitre() ?></title>
```

insère donc dynamiquement le titre défini par l'objet `Page`.

Le titre n'est pas codé en dur dans `interface.php`.

---

# 9. Construction du `<head>`

Le contenu du `<head>` est généré par :

```php
<?= $html->head() ?>
```

La méthode `head()` constitue le point central de la classe `InterfaceHtml`.

Elle regroupe plusieurs éléments :

```php
return implode("\n", [

    $this->csrf(),
    $this->bootstrap(),
    $this->menuVertical(),
    $this->menuHorizontal(),
    $this->styles(),
    $this->scripts(),
    $this->scriptPage(),
    $this->donnees()
]);
```

Le `<head>` est donc construit à partir de plusieurs méthodes spécialisées.

---

# 10. Ordre de construction du `<head>`

L'ordre actuel est :

```text
head()
 │
 ├── csrf()
 │
 ├── bootstrap()
 │
 ├── menuVertical()
 │
 ├── menuHorizontal()
 │
 ├── styles()
 │
 ├── scripts()
 │
 ├── scriptPage()
 │
 └── donnees()
```

Cet ordre est important car le HTML généré est assemblé dans cet ordre.

---

# 11. Token CSRF

La méthode :

```php
private function csrf(): string
```

gère l'ajout éventuel du token CSRF.

Elle commence par :

```php
if (!$this->page->necessiteUnJeton()) {
    return '';
}
```

Si la page n'a pas besoin de protection CSRF, aucun élément n'est ajouté.

Si elle en a besoin :

```php
return sprintf(
    '<meta name="csrf-token" content="%s">',
    Jeton::creer()
);
```

Un élément `<meta>` est alors placé dans le `<head>`.

Le JavaScript peut ensuite récupérer ce token dans la page HTML.

---

# 12. Ressources Bootstrap et CSS global

La méthode :

```php
private function bootstrap(): string
```

génère actuellement :

```html
<link rel="stylesheet"
      href="/composant/bootstrap/bootstrap.min.css">

<script src="/composant/bootstrap/bootstrap.bundle.min.js"></script>

<link rel="stylesheet"
      href="/css/style.css">
```

Cette méthode ne se limite donc pas à Bootstrap.

Elle charge également la feuille de style globale :

```text
/css/style.css
```

On peut considérer cette méthode comme le chargement des **ressources CSS et JavaScript communes**.

---

# 13. Feuilles de style spécifiques à la page

La méthode :

```php
private function styles(): string
```

récupère les styles déclarés dans l'objet `Page` :

```php
$this->page->getStyles()
```

Pour chaque fichier :

```php
foreach ($this->page->getStyles() as $style)
```

elle génère :

```html
<link rel="stylesheet" href="...">
```

Cela permet à chaque page de déclarer ses propres feuilles de style sans modifier `interface.php`.

Exemple conceptuel :

```php
$page->ajouterStyle('/css/coureur.css');
```

puis `InterfaceHtml` génère automatiquement :

```html
<link rel="stylesheet" href="/css/coureur.css">
```

---

# 14. Scripts déclarés dans `Page`

La méthode :

```php
private function scripts(): string
```

fonctionne selon le même principe pour les fichiers JavaScript.

Elle utilise :

```php
$this->page->getScripts()
```

et génère pour chaque script :

```html
<script src="..."></script>
```

Les scripts communs et les scripts propres à une page sont donc séparés.

---

# 15. Script JavaScript automatique de la page

Une particularité importante de `InterfaceHtml` est la méthode :

```php
private function scriptPage(): string
```

Elle recherche automatiquement un fichier JavaScript portant le même nom que la page PHP.

Le répertoire de la page est obtenu avec :

```php
$this->repertoirePage = dirname($_SERVER['SCRIPT_FILENAME']);
```

et son nom avec :

```php
$this->nomPage =
    pathinfo($_SERVER['PHP_SELF'], PATHINFO_FILENAME);
```

La méthode construit ensuite :

```php
$fichier =
    $this->repertoirePage . '/' .
    $this->nomPage . '.js';
```

Par exemple, si la page est :

```text
coureur.php
```

la classe recherche automatiquement :

```text
coureur.js
```

dans le même répertoire.

---

# 16. Vérification de l'existence du JavaScript

Le fichier JavaScript n'est ajouté que s'il existe :

```php
if (!is_file($fichier)) {
    return '';
}
```

Il est donc possible d'avoir une page sans JavaScript spécifique.

Cela évite d'avoir à déclarer manuellement systématiquement le script de chaque page.

---

# 17. Gestion du cache du JavaScript

Le script automatique est chargé avec :

```php
?t=<?= filemtime(...) ?>
```

Plus précisément :

```php
return sprintf(
    '<script type="module" src="%s?t=%s"></script>',
    basename($fichier),
    filemtime($fichier)
);
```

La date de modification du fichier est ajoutée à l'URL.

Exemple :

```text
coureur.js?t=1723456789
```

Lorsque le fichier JavaScript est modifié, sa date de modification change.

Le navigateur reçoit alors une nouvelle URL et est moins susceptible d'utiliser une ancienne version mise en cache.

---

# 18. Données PHP disponibles en JavaScript

La méthode :

```php
private function donnees(): string
```

permet de transmettre des données PHP au JavaScript.

Elle utilise :

```php
$this->page->getDonnees()
```

Chaque donnée possède :

- un identifiant ;
- une valeur.

La valeur est convertie en JSON :

```php
$json = ReponseJson::encoderPourHtml($valeur);
```

Puis elle est placée dans un élément :

```html
<script type="application/json" id="...">
    ...
</script>
```

Le JavaScript peut ensuite récupérer cet élément par son `id`.

---

# 19. Exemple de transmission de données

Conceptuellement, une donnée PHP pourrait être :

```php
[
    'id' => 123,
    'nom' => 'Dupont'
]
```

Elle peut être rendue disponible dans la page sous forme JSON :

```html
<script type="application/json" id="coureur">
{"id":123,"nom":"Dupont"}
</script>
```

Le JavaScript peut alors lire ce contenu et le convertir en objet JavaScript.

La conversion et l'encodage sont délégués à :

```php
ReponseJson::encoderPourHtml()
```

---

# 20. Menu vertical global

La méthode :

```php
private function menuVertical(): string
```

recherche :

```text
config/menuvertical.json
```

à partir de la racine du projet :

```php
DOSSIER_RACINE . '/config/menuvertical.json'
```

Si le fichier n'existe pas :

```php
return '';
```

Si sa lecture échoue :

```php
return '';
```

Sinon, son contenu JSON est injecté dans un module JavaScript.

Le code généré utilise :

```javascript
initialiserMenuVertical(...)
```

depuis :

```text
/composant/menuvertical/menu.js
```

---

# 21. Menu horizontal du module

La méthode :

```php
private function menuHorizontal(): string
```

recherche :

```text
../config/menuhorizontal.json
```

par rapport au répertoire de la page.

Cela permet à chaque module de disposer de son propre menu horizontal.

Si le fichier n'existe pas ou ne peut pas être lu, aucun menu horizontal n'est généré.

Sinon, le fichier JavaScript :

```text
/composant/menuhorizontal/menu.js
```

est chargé sous forme de module et appelle :

```javascript
initialiserMenuHorizontal(...)
```

---

# 22. Header global

La méthode publique :

```php
public function header(): string
```

charge :

```text
view/header.php
```

avec :

```php
return $this->chargerFragment(
    DOSSIER_RACINE . '/view/header.php',
    ['page' => $this->page]
);
```

Le fichier `header.php` reçoit donc la variable :

```php
$page
```

correspondant à la page courante.

Cela permet au header global d'utiliser les informations de l'objet `Page`.

---

# 23. Contenu spécifique de la page

La méthode :

```php
public function contenu(): string
```

construit automatiquement le nom du template HTML de la page :

```php
$this->repertoirePage . '/' .
$this->nomPage . '.html'
```

Par exemple :

```text
coureur.php
coureur.html
coureur.js
```

La page PHP peut donc être associée automatiquement à :

```text
coureur.html
```

Le template HTML est ensuite chargé par :

```php
$this->chargerFragment(...)
```

---

# 24. Footer global

La méthode :

```php
public function footer(): string
```

charge :

```text
view/footer.php
```

Le footer est donc commun aux différentes pages de l'application.

---

# 25. Chargement des fragments PHP

La méthode :

```php
private function chargerFragment(
    string $fichier,
    array $variables = []
): string
```

est une méthode interne importante de la classe.

Elle permet de charger un fichier PHP et de récupérer le HTML produit sous forme de chaîne.

Elle vérifie d'abord que le fichier existe :

```php
if (!is_file($fichier)) {
    return '';
}
```

Puis elle rend disponibles les variables :

```php
extract($variables);
```

Le tampon de sortie est ensuite activé :

```php
ob_start();
```

Le fichier est exécuté :

```php
require $fichier;
```

Et le HTML généré est récupéré :

```php
return ob_get_clean() ?: '';
```

---

# 26. Principe de `chargerFragment()`

Le mécanisme peut être représenté ainsi :

```text
fichier PHP
     │
     ▼
extract($variables)
     │
     ▼
ob_start()
     │
     ▼
require fichier
     │
     ▼
HTML généré
     │
     ▼
ob_get_clean()
     │
     ▼
string retournée
```

Cela permet à `InterfaceHtml` de construire une page à partir de plusieurs fragments indépendants.

---

# 27. Construction complète d'une page

Lorsque `interface.php` est exécuté, le fonctionnement général est :

```text
$page
  │
  ▼
new InterfaceHtml($page)
  │
  ├─────────────────────────────────┐
  │                                 │
  ▼                                 ▼
$html->head()                  HTML global
  │
  ├── CSRF
  ├── Bootstrap
  ├── style.css
  ├── menu vertical
  ├── menu horizontal
  ├── styles Page
  ├── scripts Page
  ├── script automatique
  └── données JavaScript
  │
  ▼
$html->header()
  │
  ▼
view/header.php
  │
  ▼
$html->contenu()
  │
  ▼
page.html
  │
  ▼
$html->footer()
  │
  ▼
view/footer.php
```

---

# 28. Structure finale produite

Le résultat final est conceptuellement :

```html
<!DOCTYPE html>

<html lang="fr">

<head>

    <meta charset="utf-8">

    <meta name="viewport"
          content="width=device-width, initial-scale=1">

    <title>...</title>

    <!-- Généré par InterfaceHtml -->
    ...

</head>

<body>

    <!-- header.php -->
    ...

    <main>

        <!-- page.html -->
        ...

    </main>

    <!-- footer.php -->
    ...

</body>

</html>
```

---

# 29. Convention de fichiers

La classe `InterfaceHtml` impose plusieurs conventions de nommage et d'organisation.

Pour une page appelée :

```text
coureur.php
```

elle peut rechercher automatiquement :

```text
coureur.html
coureur.js
```

dans le même répertoire.

L'application utilise également des ressources communes telles que :

```text
view/
├── header.php
└── footer.php

config/
├── menuvertical.json
└── ...

composant/
├── bootstrap/
├── menuvertical/
└── menuhorizontal/
```

Cette organisation permet de séparer :

- la structure générale ;
- le contenu des pages ;
- les scripts ;
- les styles ;
- les composants ;
- les menus ;
- les configurations.

---

# 30. Responsabilités de chaque fichier

## `interface.php`

Responsable de la structure HTML générale :

```text
<!DOCTYPE html>
<html>
<head>
<body>
<main>
```

Il orchestre l'affichage avec :

```php
$html->head()
$html->header()
$html->contenu()
$html->footer()
```

## `InterfaceHtml.php`

Responsable de la construction dynamique :

```text
head()
├── csrf()
├── bootstrap()
├── menuVertical()
├── menuHorizontal()
├── styles()
├── scripts()
├── scriptPage()
└── donnees()

header()
contenu()
footer()
```

---

# 31. Principe architectural

Cette organisation permet de séparer clairement les responsabilités.

```text
interface.php
     │
     │ structure HTML
     ▼
InterfaceHtml
     │
     │ construction dynamique
     ▼
Page + fichiers de configuration
     │
     ├── styles
     ├── scripts
     ├── données
     ├── menus
     └── fragments
```

`interface.php` ne connaît donc pas les détails de construction des menus, des scripts ou des données.

De même, `InterfaceHtml` ne contient pas le contenu métier des pages.

---

# 32. Résumé

|Élément|Responsabilité|
|---|---|
|`interface.php`|Définit le squelette HTML de la page|
|`InterfaceHtml`|Construit dynamiquement l'interface|
|`Page`|Fournit les informations propres à la page|
|`header.php`|Header global|
|`footer.php`|Footer global|
|`page.html`|Contenu HTML spécifique|
|`page.js`|JavaScript automatiquement associé|
|`menuvertical.json`|Configuration du menu vertical|
|`menuhorizontal.json`|Configuration du menu horizontal|
|`config/`|Configurations de l'application|
|`ReponseJson`|Encode les données destinées au JavaScript|
|`Jeton`|Génère le token CSRF|

---

# 33. À retenir

Le fonctionnement repose sur une idée simple :

> **`interface.php` fournit le squelette de la page, tandis que `InterfaceHtml` construit automatiquement les éléments qui remplissent ce squelette.**

Une page applicative n'a donc pas besoin de reconstruire elle-même :

- son `<head>` ;
- le header ;
- le footer ;
- les menus ;
- les ressources CSS ;
- les scripts ;
- son script JavaScript automatique ;
- les données destinées au JavaScript ;
- le token CSRF.

Elle fournit principalement son objet `Page` et son contenu propre.

La classe `InterfaceHtml` applique ensuite les conventions communes du framework pour produire la page HTML finale.