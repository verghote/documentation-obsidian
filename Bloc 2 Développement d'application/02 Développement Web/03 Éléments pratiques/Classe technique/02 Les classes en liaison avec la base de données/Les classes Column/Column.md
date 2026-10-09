Voici la version revue et complétée de la documentation. Elle intègre la notion de **nettoyage / assainissement (`sanitize`)**, clarifie le cycle de validation et met à jour les exemples pour refléter la nouvelle architecture.

# Documentation : La classe `Column`

## Présentation

La classe `Column` est la classe de base utilisée pour représenter une colonne métier persistée dans une classe métier.

Une colonne métier associe :

- une valeur ;
    
- **un traitement de nettoyage (assainissement) ;**
    
- des règles de validation ;
    
- des informations nécessaires aux opérations CRUD.
    

Elle constitue le lien entre :

- la classe métier représentant une table ;
    
- les données manipulées par l'application ;
    
- la base de données.
    

Les classes `Column` sont utilisées dans les classes métier qui héritent généralement de `Table`.

PHP

```
class Categorie extends Table
{
    protected function defineColumns(): void
    {
        $nom = new ColumnText(
            required: true,
            maxLength: 50
        );

        $this->addColumn('nom', $nom);
    }
}
```

_Ici, la colonne SQL `nom` est représentée par un objet `ColumnText` qui hérite de `Column`._

## Rôle de `Column`

`Column` est une classe **abstraite**. Elle ne doit jamais être instanciée directement. Elle fournit le comportement commun à toutes les colonnes :

- gestion de la valeur ;
    
- **nettoyage et assainissement des données entrée (`sanitize`) ;**
    
- gestion du caractère obligatoire ;
    
- contrôle avant insertion ou modification ;
    
- gestion des messages d'erreur.
    

Les classes spécialisées héritent de `Column` :

Plaintext

```
Column
 |
 +-- ColumnText
 |
 +-- ColumnTextarea
 |
 +-- ColumnInt
 |
 +-- ColumnDate
 |
 +-- ...
```

Chaque classe fille ajoute ses propres traitements et contrôles :

- `ColumnText` nettoie le texte (titre, casse, espaces, accents) puis valide le format (regex, longueur) ;
    
- `ColumnTextarea` filtre ou encode le contenu HTML ;
    
- `ColumnInt` contrôle qu'une valeur est un entier.
    

## La propriété `$Value`

PHP

```
public mixed $Value;
```

Cette propriété contient la valeur associée à la colonne.

PHP

```
$age = new ColumnInt();
$age->Value = 25;
```

La valeur est ensuite utilisée par les opérations CRUD. Lors d'une modification :

PHP

```
$categorie->setValue('ageMin', 10);
```

la valeur est stockée dans la colonne correspondante.

## La méthode `sanitize()` (Nettoyage de la donnée)

PHP

```
public function sanitize(mixed $value): mixed
```

Cette méthode permet de **nettoyer et normaliser la donnée** avant d'effectuer les contrôles de validité.

Par défaut, dans la classe abstraite `Column`, la méthode retourne la valeur inchangée :

PHP

```
public function sanitize(mixed $value): mixed
{
    return $value;
}
```

Les classes filles redéfinissent cette méthode pour appliquer leurs règles d'assainissement spécifiques **avant** validation.

### Exemples de surcharges :

- **`ColumnText`** : applique un nettoyage de type titre (`Std::nettoyerTitre`), transforme la casse (`UPPER`, `Lower`, `First`...), retire les accents ou réduit les espaces multiples selon sa configuration.
    
- **`ColumnTextarea`** : nettoie le code HTML (`Std::nettoyerHtml`), supprime les balises interdites ou applique un `htmlspecialchars`.
    

## La propriété `$Required`

PHP

```
public readonly bool $Required;
```

Indique si la colonne doit obligatoirement contenir une valeur.

Par défaut :

PHP

```
new ColumnText();
```

équivaut à :

PHP

```
new ColumnText(required: true);
```

### Comportement :

- **Une colonne obligatoire** (`required: true`) refusera `null` ou une chaîne ne contenant que des espaces `" "`.
    
- **Une colonne facultative** (`required: false`) acceptera une valeur vide ou `null`.
    

## Les propriétés CRUD

### `Insertable`

PHP

```
public readonly bool $Insertable;
```

Indique si la colonne peut être utilisée lors d'une insertion (`INSERT`).

_Exemple — Une clé primaire auto-incrémentée :_

PHP

```
$id = new ColumnInt(insertable: false);
```

La colonne existe dans la table mais ne sera pas transmise dans la requête `INSERT`.

### `Updatable`

PHP

```
public readonly bool $Updatable;
```

Indique si la colonne peut être modifiée (`UPDATE`).

_Exemple — Une date de création :_

PHP

```
$dateCreation = new ColumnDate(
    insertable: true,
    updatable: false
);
```

La valeur est enregistrée à la création mais ne pourra plus être modifiée par la suite.

## La méthode `checkValidity()` et cycle de validation

PHP

```
public function checkValidity(): bool
```

Cette méthode réalise les contrôles de la colonne. Son exécution suit un ordre strict :

1. **Nettoyage automatique (`sanitize`)** : la valeur présente dans `$Value` est immédiatement nettoyée et réaffectée à `$Value`.
    
2. **Contrôle d'obligation (`Required`)** : elle vérifie qu'une valeur obligatoire est bien renseignée.
    

### Implémentation dans `Column` :

PHP

```
public function checkValidity(): bool
{
    // 1. Assainissement préalable de la donnée
    if ($this->Value !== null) {
        $this->Value = $this->sanitize($this->Value);
    }

    // 2. Contrôle du caractère obligatoire
    if ($this->Required && ($this->Value === null || strlen(trim((string)$this->Value)) === 0)) {
        $this->validationMessage = "Veuillez renseigner ce champ.";
        return false;
    }

    return true;
}
```

### Redéfinition dans les classes filles :

Les classes filles redéfinissent `checkValidity()` pour exécuter leurs propres règles de validation **sur la valeur nettoyée**. Elles commencent toujours par appeler `parent::checkValidity()`.

PHP

```
class ColumnText extends Column
{
    public function checkValidity(): bool
    {
        // Exécute d'abord sanitize() et le contrôle Required
        if (!parent::checkValidity()) {
            return false;
        }

        // Si le champ facultatif est vide, la validation réussit
        if ($this->Value === null || $this->Value === '') {
            return true;
        }

        // Ici, $this->Value est DEJA nettoyée/assainie
        // Exécution des règles spécifiques (Pattern, MinLength, MaxLength)
        if ($this->Pattern !== null && !preg_match($this->Pattern, (string)$this->Value)) {
            $this->validationMessage = "La valeur transmise n'est pas valide.";
            return false;
        }

        return true;
    }
}
```

> **Important** : Grâce à cet ordre d'exécution, la validation par expression régulière (`Pattern`) ou par longueur (`MinLength`, `MaxLength`) s'applique toujours sur la chaîne **déjà assainie et transformée**.

## Récupérer un message d'erreur

Lorsqu'une validation échoue, le message associé s'obtient avec `getValidationMessage()` :

PHP

```
if (!$colonne->checkValidity()) {
    echo $colonne->getValidationMessage();
}
```

## Exemple complet dans une classe métier

PHP

```
protected function defineColumns(): void
{
    // Champ texte obligatoire, nettoyé (espaces/casse) et limité à 30 caractères
    $colonne = new ColumnText(
        required: true,
        maxLength: 30,
        casse: TextCase::First,
        supprimerEspaceSuperflu: true
    );
    $this->addColumn('nom', $colonne);

    // Champ entier obligatoire avec plage de valeurs
    $colonne = new ColumnInt(
        required: true,
        min: 1,
        max: 99
    );
    $this->addColumn('age', $colonne);
}
```

## À retenir

`Column` représente une donnée persistée dans l'application. Elle permet de centraliser :

1. **La structure des données** ;
    
2. **Le nettoyage et la normalisation des saisies (`sanitize`)** ;
    
3. **Les règles de validation (`checkValidity`)** ;
    
4. **Les contraintes CRUD (`Required`, `Insertable`, `Updatable`)**.
    

Les classes métier et les contrôleurs n'ont plus à nettoyer les données manuellement : le nettoyage et la validation sont totalement délégués aux objets `Column`.