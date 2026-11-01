# 1. Installation de PHPUnit avec Composer

PHPUnit est installé comme dépendance de développement avec Composer :

```bash
composer require --dev phpunit/phpunit
````

L'option `--dev` indique que PHPUnit est utilisé uniquement pendant le développement et ne sera pas nécessaire en production.

La commande suivante permet de vérifier l'installation :

```bash
vendor/bin/phpunit --version
```

L'exécution des tests se fait ensuite avec :

```shell
vendor/bin/phpunit
```

# 2. Organisation des tests dans un projet

Les tests sont généralement regroupés dans un répertoire dédié :

```
projet/
│
├── src/
│   └── ClasseMetier/
│       └── Club.php
│
├── tests/
│   └── ClasseMetier/
│       └── ClubTest.php
│
├── composer.json
└── phpunit.xml
```

Principe :

- le dossier `src` contient le code de l'application ;
- le dossier `tests` contient les classes de tests associées.

Une classe test porte généralement le même nom que la classe testée avec le suffixe `Test`.

Exemple :

```
Club.php
ClubTest.php
```

# 3. Écriture d'un test simple

Un test PHPUnit est une classe qui hérite de :

```php
PHPUnit\Framework\TestCase
```

Exemple :

```php
use PHPUnit\Framework\TestCase;

class ClubTest extends TestCase
{
    public function testNomValide(): void
    {
        $club = new Club();

        $resultat = $club->validerNom("Paris");

        $this->assertTrue($resultat);
    }
}
```

Un test vérifie toujours :

- une situation donnée ;
- une action réalisée ;
- un résultat attendu.

---

# 4. Principales assertions PHPUnit

Les assertions permettent de comparer le résultat obtenu avec le résultat attendu.

## Vérifier une valeur booléenne

```
$this->assertTrue($resultat);

$this->assertFalse($resultat);
```

---

## Vérifier une valeur

```
$this->assertEquals(
    "PARIS",
    $club->getNom()
);
```

---

## Vérifier une valeur identique

```
$this->assertSame(
    10,
    $age
);
```

`assertSame()` vérifie également le type.

---

## Vérifier une valeur nulle

```
$this->assertNull($logo);
```

---

## Vérifier une exception

```
$this->expectException(Exception::class);

$objet->methodeIncorrecte();
```

---

# 5. Tester les succès et les erreurs

Un test ne doit pas uniquement vérifier les cas qui fonctionnent.

Il doit également vérifier les comportements incorrects.

Exemple : validation d'un nom de club.

## Cas valide

```
public function testNomCorrect(): void
{
    $club = new Club();

    $this->assertTrue(
        $club->validerNom("Paris")
    );
}
```

---

## Cas invalide

```
public function testNomVide(): void
{
    $club = new Club();

    $this->assertFalse(
        $club->validerNom("")
    );
}
```

Il faut donc prévoir :

- les données normales ;
- les valeurs limites ;
- les données incorrectes ;
- les erreurs attendues.

---

# 6. Utilisation des Data Providers

Lorsqu'une même méthode doit être testée avec plusieurs valeurs, un **Data Provider** évite de recopier plusieurs tests.

Exemple :

```
/**
 * @dataProvider nomsInvalides
 */
public function testNomInvalide(string $nom): void
{
    $club = new Club();

    $this->assertFalse(
        $club->validerNom($nom)
    );
}


public static function nomsInvalides(): array
{
    return [
        [''],
        ['123'],
        ['@test'],
        ['Nom trop long']
    ];
}
```

Le même test est exécuté plusieurs fois avec des données différentes.

Avantages :

- moins de duplication ;
- tests plus lisibles ;
- ajout simple de nouveaux cas.

---

# Résumé

PHPUnit permet de vérifier automatiquement le comportement d'une application.

Les bonnes pratiques sont :

- installer PHPUnit avec Composer ;
- séparer le code et les tests ;
- créer un test par comportement attendu ;
- tester les succès et les erreurs ;
- utiliser les assertions adaptées ;
- utiliser les Data Providers pour tester plusieurs jeux de données sans duplication.