# Présentation

Les **PSR** (*PHP Standards Recommendations*) sont des recommandations publiées par le **PHP-FIG** (*PHP Framework Interoperability Group*).

Leur objectif est de définir des règles communes afin que les bibliothèques et les frameworks PHP puissent fonctionner ensemble.

Respecter les PSR permet notamment de :

- produire un code plus lisible ;
- faciliter le travail en équipe ;
- rendre les projets plus homogènes ;
- utiliser facilement des bibliothèques externes.

# Les PSR essentielles

Dans la pratique, quelques PSR seulement sont réellement indispensables.

| PSR    | Sujet                      | À connaître ? |
| ------ | -------------------------- | :-----------: |
| PSR-1  | Règles de base du code PHP |     ⭐⭐⭐⭐⭐     |
| PSR-4  | Autoloading des classes    |     ⭐⭐⭐⭐⭐     |
| PSR-12 | Style d'écriture du code   |     ⭐⭐⭐⭐⭐     |
| PSR-3  | Journalisation (Logger)    |     ⭐⭐⭐☆☆     |
| PSR-7  | Requêtes et réponses HTTP  |     ⭐⭐⭐☆☆     |

Les trois premières sont incontournables.

# PSR-1 — Basic Coding Standard

Cette norme définit les règles générales d'écriture.

## Chaque fichier doit utiliser une seule convention

Exemple :

```php
<?php
declare(strict_types=1);
```

## Les classes utilisent le PascalCase

Correct :

```php
class Client
```

Incorrect :

```php
class client
```

## Les méthodes utilisent le camelCase

Correct :

```php
public function getNom()
```

Incorrect :

```php
public function GetNom()
```

## Les constantes utilisent les majuscules

Correct :

```php
public const MAX_SIZE = 100;
```

## Les fichiers ne doivent produire aucun effet de bord

Un fichier contenant une classe ne doit pas exécuter de traitement.

Correct :

```php
class Client
{
}
```

À éviter :

```php
echo "Bonjour";
```

ou

```php
Database::connect();
```

dans un fichier contenant uniquement une classe.

---

# PSR-4 — Autoloading

C'est probablement la norme la plus importante.

Elle définit comment retrouver automatiquement une classe à partir de son namespace.

Exemple :

```php
namespace ClasseTechnique;

class Requete
{
}
```

doit être stockée dans

```
ClasseTechnique/Requete.php
```

ou

```
src/ClasseTechnique/Requete.php
```

selon la configuration de l'autoloader.

Grâce à PSR-4 :

```php
use ClasseTechnique\Requete;

Requete::verifierPost();
```

charge automatiquement :

```
ClasseTechnique/Requete.php
```

sans faire de `require`.

# PSR-12 — Extended Coding Style

PSR-12 définit la manière d'écrire le code.

C'est aujourd'hui le standard le plus utilisé.

## Ordre des déclarations

```php
<?php
declare(strict_types=1);

namespace App;

use PDO;
use Exception;
```

## Une accolade ouvrante sur une nouvelle ligne

Correct :

```php
class Client
{
}
```

Correct :

```php
public function getNom(): string
{
}
```

## Une instruction par ligne

Correct :

```php
$a = 10;
$b = 20;
```

À éviter :

```php
$a = 10; $b = 20;
```

## Indentation

Utiliser **4 espaces**.

Ne pas utiliser les tabulations.

## Longueur des lignes

Il est conseillé de rester autour de 120 caractères maximum.

## Espaces

Correct :

```php
if ($age > 18) {
```

Incorrect :

```php
if($age>18){
```

## Visibilité explicite

Toujours écrire :

```php
public
protected
private
```

Ne jamais omettre la visibilité.
## Types

Toujours typer les paramètres :

```php
public function ajouter(string $nom): void
```

plutôt que

```php
public function ajouter($nom)
```

# PSR-3 — Logger

Cette norme définit une interface commune de journalisation.

Les niveaux sont :

```
emergency
alert
critical
error
warning
notice
info
debug
```

Exemple :

```php
$logger->error("Connexion impossible.");
```

Toutes les bibliothèques compatibles PSR-3 fonctionnent de la même manière.

# PSR-7 — Requêtes HTTP

PSR-7 représente une requête HTTP sous forme d'objet.

Exemple :

```php
$request->getMethod();

$request->getQueryParams();

$request->getParsedBody();
```

Les frameworks modernes (Slim, Laminas, Mezzio...) utilisent PSR-7.

# Les PSR les plus utilisées

Dans la majorité des projets PHP modernes, on rencontre principalement :

- PSR-1
- PSR-4
- PSR-12

Ces trois normes couvrent déjà :

- l'organisation des classes ;
- le chargement automatique ;
- le style d'écriture.

# Dans un framework personnel

Même sans utiliser Composer, il est recommandé de respecter :

- les namespaces (PSR-4) ;
- un autoloader compatible PSR-4 ;
- les conventions d'écriture de PSR-12.

Cela rendra le code plus facile à maintenir et facilitera une éventuelle migration vers un framework plus important.

# Bonnes pratiques complémentaires

## Déclarer les types stricts

```php
declare(strict_types=1);
```

## Utiliser les namespaces

```php
namespace ClasseTechnique;
```

## Importer les classes avec `use`

```php
use ClasseTechnique\Requete;
use ClasseTechnique\Jeton;
```

## Éviter les variables globales

Préférer :

```php
Requete::postString('nom');
```

plutôt que :

```php
$_POST['nom'];
```

## Documenter les classes publiques

Utiliser PHPDoc :

```php
/**
 * Retourne la liste des clients.
 *
 * @return array
 */
```

# Les PSR à retenir absolument

| PSR    | Pourquoi ?                                    |
| ------ | --------------------------------------------- |
| PSR-1  | Règles générales d'écriture                   |
| PSR-4  | Chargement automatique des classes            |
| PSR-12 | Présentation et style du code                 |

# Conclusion

Pour développer une application PHP moderne, il n'est pas nécessaire de connaître toutes les PSR.

La maîtrise de **PSR-1**, **PSR-4** et **PSR-12** constitue déjà une excellente base. 
Elles définissent les conventions fondamentales de nommage, d'organisation des fichiers, de chargement automatique des classes et de style de code. Les autres PSR deviennent utiles au fur et à mesure que l'on utilise des bibliothèques ou des frameworks plus avancés.
