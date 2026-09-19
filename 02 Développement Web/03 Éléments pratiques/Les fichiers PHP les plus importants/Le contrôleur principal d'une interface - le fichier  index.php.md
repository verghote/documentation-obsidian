# Dans le cadre d'une consultation

```php
<?php  
declare(strict_types=1);  
  
use ClasseMetier\Categorie;  
use ClasseTechnique\Page;  
  
require $_SERVER['DOCUMENT_ROOT'] . "/../bootstrap/autoload.php";  
  
// alimentation et affichage de l'interface  
// chargement des catégories et du composant assurant la génération d'un document PDF à partir de la page HTML  
$page = new Page();  
$page->setDonnee("lesCategories", Categorie::getAll())  
    ->afficher();
```


Dans le cadre d'un ajout

```php
<?php  
declare(strict_types=1);  
  
use ClasseMetier\Categorie;  
use ClasseTechnique\Page;  
  
// activation du chargement dynamique des ressources  
require $_SERVER['DOCUMENT_ROOT'] . "/../bootstrap/autoload.php";  
  
// alimentation et affichage de l'interface  
// chargement des catégories et du composant assurant la génération d'un document PDF à partir de la page HTML  
$page = new Page();  
$page->setDonnee("lesCategories", Categorie::getAll())  
    ->avecJeton()  
    ->afficher();
```

Dans le cadre d'une modification

```php
<?php  
declare(strict_types=1);  
  
use ClasseMetier\Categorie;  
use ClasseTechnique\Page;  
  
// activation du chargement dynamique des ressources  
require $_SERVER['DOCUMENT_ROOT'] . "/../bootstrap/autoload.php";  
  
// alimentation et affichage de l'interface  
$page = new Page();  
$page->setDonnee("lesCategories", Categorie::getAll())  
    ->avecJeton()  
    ->afficher();
```