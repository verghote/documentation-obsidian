# Rôle de la classe

`Table` est la classe technique centrale du framework de persistance.

Elle fournit un moteur générique permettant de manipuler les tables SQL sans que les classes métier aient à écrire les opérations courantes d'accès aux données.

Son rôle est de factoriser toute la mécanique commune :

- connexion à la base de données ;
- gestion des colonnes métier ;
- validation générique des données ;
- gestion des erreurs de validation ;
- contrôle des contraintes d'unicité ;
- génération automatique des requêtes SQL ;
- exécution des opérations CRUD.

Les classes métier ne décrivent donc pas **comment** effectuer les opérations sur la base de données, mais uniquement **ce qui caractérise leur table**.

# Philosophie

Le framework repose sur un principe simple :

> Une classe métier décrit sa structure.  
> La classe `Table` déduit automatiquement le comportement à adopter.

Une classe métier ne construit jamais ses requêtes SQL d'insertion ou de modification.

Elle décrit uniquement :

- le nom de la table SQL ;
- sa clé primaire ;
- les colonnes manipulées ;
- leurs règles de validation ;
- leurs autorisations d'insertion et de modification ;
- les contraintes d'unicité ;
- les éventuelles règles métier.

À partir de cette description, `Table` est capable de :

- contrôler les données reçues ;
- détecter les colonnes interdites ;
- vérifier les champs obligatoires ;
- contrôler les contraintes d'unicité ;
- construire automatiquement les requêtes `INSERT` et `UPDATE`.

Cette approche permet d'obtenir des classes métier courtes, homogènes et faciles à maintenir.

# Principe d'utilisation

Chaque table applicative est représentée par une classe héritant de `Table`.

Exemple :

```php
class Club extends Table
{
    protected function configure(): void
    {
        $this->table = 'club';
        $this->primaryKey = 'id';

        // Déclaration des colonnes...

        // Déclaration des contraintes...
    }
}
```

Une classe métier ne contient généralement que :

- sa configuration ;
- quelques méthodes de consultation ;
- les éventuelles règles métier spécifiques.

Toutes les opérations CRUD sont héritées de `Table`.

# Cycle de vie d'un objet

Lorsqu'une classe métier est instanciée, le constructeur de `Table` réalise automatiquement toutes les opérations techniques nécessaires.

Le cycle est le suivant :

```text
new ClasseMetier()

        ▼
constructeur Table()
        ▼
Connexion à la base de données
        ▼
configure()
        ▼
validateConfiguration()
        ▼
Objet prêt à être utilisé
```

À l'issue de ce processus :

- la connexion PDO est disponible ;
- la table SQL est connue ;
- la clé primaire est définie ;
- les colonnes sont enregistrées ;
- les contraintes d'unicité sont déclarées.

L'objet est alors prêt à réaliser des opérations de consultation, d'ajout, de modification ou de suppression.

# Ce que connaît la classe `Table`

La classe `Table` est volontairement générique.

Elle ne connaît jamais :

- les tables métier ;
- les règles fonctionnelles de l'application ;
- les relations entre les objets métier.

Elle connaît uniquement :

- une connexion PDO ;
- une collection d'objets `Column` ;
- une liste de contraintes d'unicité ;
- les règles génériques permettant d'effectuer les opérations CRUD.

Toutes les connaissances métier sont fournies par les classes dérivées lors de leur configuration.

# Responsabilités respectives

La répartition des responsabilités est la suivante :

| Classe | Responsabilité |
|---------|----------------|
| `Table` | Validation générique, génération des requêtes SQL, gestion des erreurs, contrôle d'unicité, opérations CRUD |
| `Column` | Validation d'une valeur individuelle (type, format, longueur, etc.) |
| Classe métier | Description de la table, règles métier spécifiques, consultations particulières |
| `Database` | Fourniture de la connexion PDO |

Cette séparation permet de maintenir un framework générique tout en laissant chaque classe métier exprimer uniquement les règles propres à son domaine fonctionnel.

# Configuration d'une classe métier

Chaque classe métier héritant de `Table` doit fournir une description complète de la table SQL qu'elle représente.

Cette description est réalisée dans la méthode :

```php
protected function configure(): void
```

Cette méthode constitue le point d'entrée principal de la classe métier.

Elle doit définir :

- le nom de la table SQL ;
- la clé primaire ;
- les colonnes manipulées par l'application ;
- les contraintes d'unicité éventuelles.

La classe `Table` utilise ensuite cette configuration pour assurer automatiquement les contrôles et les opérations CRUD.

# Méthode `configure()`

La méthode `configure()` est abstraite dans `Table`.

Chaque classe fille a donc l'obligation de l'implémenter.

Exemple :

```php
class Club extends Table
{
    protected function configure(): void
    {
        $this->table = 'club';

        $this->primaryKey = 'id';

        $this->addColumn(
            'nom',
            new ColumnText(...)
        );
    }
}
```

La classe métier ne doit pas :

- ouvrir de connexion SQL ;
- écrire de requête `INSERT` ;
- écrire de requête `UPDATE` ;
- gérer les erreurs techniques.

Elle doit uniquement décrire son modèle.

# Définition du nom de table

La propriété :

```php
protected string $table;
```

contient le nom réel de la table SQL.

Exemple :

```php
$this->table = 'club';
```

Cette information est utilisée par toutes les opérations SQL :

- recherche d'existence ;
- chargement d'un enregistrement ;
- insertion ;
- modification ;
- suppression.

Le nom de table doit correspondre exactement à celui défini dans la base de données.

# Définition de la clé primaire

La propriété :

```php
protected string $primaryKey;
```

indique la colonne utilisée comme identifiant unique d'un enregistrement.

Exemple :

```php
$this->primaryKey = 'id';
```

La clé primaire est utilisée automatiquement pour :

- vérifier l'existence d'un enregistrement ;
- charger un enregistrement ;
- modifier un enregistrement ;
- supprimer un enregistrement ;
- exclure l'enregistrement courant lors des contrôles d'unicité.

La classe `Table` ne suppose pas que la clé primaire s'appelle `id`.

Exemples possibles :

```php
$this->primaryKey = 'licence';
```

ou :

```php
$this->primaryKey = 'reference';
```

# Déclaration des colonnes

Les colonnes sont déclarées avec :

```php
$this->addColumn(string $name, Column $column);
```

Chaque colonne est représentée par un objet `Column` ou une classe dérivée.

Exemple :

```php
	$column = new ColumnText(  
	    required: true,  
	    insertable: true,  
	    updatable: false,  
	    casse: TextCase::Upper,  
	    supprimerAccent: true,  
	    supprimerEspaceSuperflu: true,  
	    pattern: "^[A-Z]( ?[A-Z]+)*$",  
	    maxLength: 30  
	);  
	  
	$this->addColumn('nom', $column);
```

Lorsqu'une colonne est ajoutée, elle devient connue de `Table`.

Elle pourra alors participer :

- à la validation ;
- aux contrôles d'autorisation ;
- aux opérations d'insertion ;
- aux opérations de modification.

# Vérification des colonnes

La méthode :

```php
addColumn()
```

effectue également un contrôle de cohérence.

Une même colonne ne peut pas être déclarée plusieurs fois.

Exemple interdit :

```php
$this->addColumn('nom', new ColumnText());

$this->addColumn('nom', new ColumnText());
```

La seconde déclaration provoque une erreur.

Cette vérification évite les configurations ambiguës.

# Validation de configuration

Après l'appel à :

```php
configure()
```

le constructeur appelle :

```php
validateConfiguration()
```

Cette méthode vérifie que la classe métier est correctement définie.

Les contrôles réalisés sont :

- une table SQL doit être renseignée ;
- une clé primaire doit être renseignée ;
- au moins une colonne métier doit être définie.

Exemple :

```php
private function validateConfiguration(): void
{
    if ($this->table === '') {
        throw new Exception(
            "Le nom de la table n'est pas défini."
        );
    }

    if ($this->primaryKey === '') {
        throw new Exception(
            "La clé primaire n'est pas définie."
        );
    }

    if (empty($this->columns)) {
        throw new Exception(
            "Aucune colonne métier n'est définie."
        );
    }
}
```

# Objectif de cette validation

Cette étape permet de détecter rapidement :

- une nouvelle classe métier incomplète ;
- un oubli de configuration ;
- une faute de frappe dans le nom d'une table ;
- une évolution ayant rendu une classe incompatible.

Une erreur de configuration doit être détectée au démarrage de l'objet et non lors d'une opération utilisateur.

# Déclaration des contraintes d'unicité

Les contraintes d'unicité sont ajoutées avec :

```php
$this->addUniqueConstraint(ContrainteUnique $constraint);
```

Exemple :

```php
class Club extends Table
{
    protected function configure(): void
    {
        $this->table = 'club';
        $this->primaryKey = 'id';

        // Déclaration des colonnes...

        // Déclaration des contraintes
        $this->addUniqueConstraint(new ContrainteUnique(['nom'], 'Ce nom existe déjà.'));
    }
}

```

Une contrainte peut porter sur :

- une seule colonne ;

exemple :

```php
['nom']
```

- plusieurs colonnes ;

exemple :

```php
[
    'nom',
    'prenom',
    'dateNaissance'
]
```

Les contraintes composées permettent de gérer les règles d'unicité métier qui ne peuvent pas être représentées par une seule colonne.

---

# Vérification des contraintes

Lors d'un ajout ou d'une modification :

```php
checkUniqueConstraints()
```

est exécutée automatiquement.

Le traitement est :

1. récupération des valeurs actuelles ;
2. recherche d'un enregistrement possédant les mêmes valeurs ;
3. ajout d'une erreur si un doublon existe.

Lors d'une modification, l'enregistrement en cours est exclu de la recherche.

Ainsi :

```text
Modification du club 12

Nom actuel : "Valenciennes"

Recherche d'un doublon :

Nom = "Valenciennes"
ET id <> 12
```

L'objet ne se détecte donc pas lui-même comme doublon.

# Exemple complet de configuration

Une classe métier complète peut ressembler à ceci :

```php
class Club extends Table
{
    protected function configure(): void
    {
        $this->table = 'club';

        $this->primaryKey = 'id';

        $this->addColumn(
            'nom',
            new ColumnText(...)
        );

        $this->addColumn(
            'adresse',
            new ColumnText(...)
        );

        $this->addUniqueConstraint(
            new ContrainteUnique(
                ['nom'],
                'Ce club existe déjà.'
            )
        );
    }
}
```

La classe métier décrit alors entièrement son comportement sans écrire de logique SQL.

# Les colonnes

Les colonnes constituent le cœur de la description d'une table métier.

Dans le framework, une colonne SQL n'est pas représentée par un simple nom de champ, mais par un objet `Column` ou par une classe dérivée.

Une colonne contient :

- sa valeur courante ;
- ses règles de validation ;
- ses contraintes ;
- ses autorisations d'utilisation lors des opérations CRUD.

La classe `Table` ne connaît donc pas la structure détaillée des colonnes. Elle délègue cette responsabilité aux objets `Column`.


# Rôle d'un objet `Column`

Un objet `Column` représente une colonne métier manipulée par l'application.

Il est responsable de contrôler une valeur individuelle.

Selon son type, il peut vérifier :

- le type de donnée ;
- la longueur maximale ;
- le format ;
- les valeurs autorisées ;
- l'obligation de présence ;
- les règles spécifiques au champ.

Exemples de colonnes possibles :

```php
ColumnText
ColumnInteger
ColumnDate
ColumnBoolean
ColumnEnum
```

Chaque classe spécialisée apporte ses propres règles de validation.

# Déclaration d'une colonne

Une colonne est ajoutée dans la méthode `configure()` :

```php
$this->addColumn(
    'nom',
    new ColumnText(...)
);
```

La méthode `Table` enregistre alors cette colonne.

À partir de ce moment, elle participe automatiquement :

- aux contrôles de saisie ;
- aux opérations d'ajout ;
- aux opérations de modification.

# Les propriétés CRUD d'une colonne

En plus de ses règles de validation, chaque colonne possède trois propriétés essentielles qui déterminent son comportement dans les opérations CRUD :

| Propriété | Rôle |
|---|---|
| `Required` | Indique si la valeur est obligatoire |
| `Insertable` | Indique si la colonne peut être renseignée lors d'un ajout |
| `Updatable` | Indique si la colonne peut être modifiée |

Ces trois propriétés permettent à `Table` de déterminer automatiquement :

- les colonnes autorisées en insertion ;
- les colonnes obligatoires ;
- les colonnes modifiables.

# Propriété `Required`

## Définition

La propriété :

```php
Required
```

indique si une valeur est obligatoire lors d'un ajout.

Si :

```text
Required = true
```

la colonne doit obligatoirement être présente dans les données transmises à :

```php
add()
```

Exemple :

```php
[
    'nom' => 'Club A'
]
```

Si la colonne :

```text
nom
```

est obligatoire et qu'elle n'est pas fournie :

```php
[]
```

la validation échoue.

Une erreur est ajoutée :

```text
nom est obligatoire.
```

---

## Colonne obligatoire

Exemple :

```php
$this->addColumn(
    'nom',
    new ColumnText(
        required: true
    )
);
```

Cette colonne doit toujours recevoir une valeur lors d'une création.

---

## Colonne optionnelle

Si :

```text
Required = false
```

la colonne peut ne pas être présente.

Exemple :

```php
$this->addColumn(
    'telephone',
    new ColumnText(
        required: false
    )
);
```

Les deux situations sont acceptées :

```php
[
    'telephone' => '0600000000'
]
```

ou :

```php
[
    'telephone' => null
]
```

Une chaîne vide n'est pas considérée comme une valeur valide pour une colonne optionnelle.

---

# Propriété `Insertable`

## Définition

La propriété :

```php
Insertable
```

indique si la colonne peut participer à une opération d'ajout.

Si :

```text
Insertable = true
```

la colonne peut être intégrée dans une requête :

```sql
INSERT
```

Si :

```text
Insertable = false
```

elle est exclue automatiquement.

---

# Exemple : clé primaire auto-incrémentée

Une clé primaire générée par la base :

```text
id
```

ne doit généralement pas être fournie par l'utilisateur.

Configuration :

```text
Required   = false
Insertable = false
Updatable  = false
```

La base de données génère elle-même la valeur.

---

# Exemple : valeur calculée automatiquement

Une colonne :

```text
dateCreation
```

peut être alimentée automatiquement par l'application :

```text
Required   = false
Insertable = true
Updatable  = false
```

Elle est créée lors de l'ajout mais ne pourra plus être modifiée.

---

# Propriété `Updatable`

## Définition

La propriété :

```php
Updatable
```

indique si une colonne peut être modifiée.

Si :

```text
Updatable = true
```

elle participe aux requêtes :

```sql
UPDATE
```

Si :

```text
Updatable = false
```

elle est protégée contre les modifications.

---

# Exemple : date de création

Une date de création ne doit généralement jamais changer.

Configuration :

```text
dateCreation

Required   = false
Insertable = true
Updatable  = false
```

Lors d'une modification :

```php
$club->modify(
    10,
    [
        'dateCreation' => '2026-01-01'
    ]
);
```

la colonne n'est pas considérée comme modifiable.

---

# Combinaisons courantes

## Champ saisi par l'utilisateur

Exemple :

```text
nom
```

Configuration :

| Propriété | Valeur |
|-|-|
| Required | true |
| Insertable | true |
| Updatable | true |

---

## Champ optionnel

Exemple :

```text
telephone
```

Configuration :

| Propriété | Valeur |
|-|-|
| Required | false |
| Insertable | true |
| Updatable | true |

---

## Clé primaire auto-incrémentée

Exemple :

```text
id
```

Configuration :

| Propriété | Valeur |
|-|-|
| Required | false |
| Insertable | false |
| Updatable | false |

---

## Date de création automatique

Exemple :

```text
dateCreation
```

Configuration :

| Propriété | Valeur |
|-|-|
| Required | false |
| Insertable | true |
| Updatable | false |

---

## Champ technique non manipulé

Exemple :

```text
version
```

Configuration :

| Propriété | Valeur |
|-|-|
| Required | false |
| Insertable | false |
| Updatable | false |

---

# Tableau récapitulatif

| Type de colonne | Required | Insertable | Updatable |
|-|-:|-:|-:|
| Clé primaire auto-incrémentée | Non | Non | Non |
| Champ obligatoire utilisateur | Oui | Oui | Oui |
| Champ optionnel utilisateur | Non | Oui | Oui |
| Date de création automatique | Non | Oui | Non |
| Utilisateur créateur | Non | Oui | Non |
| Date de modification automatique | Non | Non | Oui |
| Champ calculé par SQL | Non | Non | Non |

---

# Interaction avec la classe `Table`

Ces propriétés sont utilisées automatiquement par les méthodes internes.

## Lors d'un ajout

`Table` utilise :

```php
getInsertColumns()
```

pour déterminer les colonnes autorisées.

Puis :

```php
getRequiredColumns()
```

pour déterminer les colonnes obligatoires.

---

## Lors d'une modification

`Table` utilise :

```php
getUpdateColumns()
```

pour déterminer les colonnes pouvant être modifiées.

---

# Principe général

La définition des colonnes constitue une description complète du comportement CRUD.

Le développeur ne doit normalement pas modifier :

- `getInsertColumns()`
- `getRequiredColumns()`
- `getUpdateColumns()`

Il doit simplement déclarer correctement ses colonnes.

La classe `Table` applique ensuite automatiquement les règles définies.

# Gestion des opérations CRUD

La classe `Table` fournit une implémentation générique des opérations de création, modification et suppression.

Les classes métier n'ont donc pas à écrire :

- les requêtes SQL ;
- la gestion des paramètres ;
- les contrôles de présence ;
- la gestion des erreurs techniques ;
- la récupération des identifiants générés.

Elles fournissent uniquement :

- les données reçues ;
- la configuration des colonnes ;
- les règles métier spécifiques.

---

# Ajout d'un enregistrement

La méthode publique :

```
public function add(array $data): bool
```

permet d'insérer un nouvel enregistrement.

Elle applique automatiquement le cycle suivant :

```
add()

   |
   v

reset()

   |
   v

getInsertColumns()

   |
   v

populateData()

   |
   v

checkColumns()

   |
   v

checkUniqueConstraints()

   |
   v

beforeChange()

   |
   v

beforeInsert()

   |
   v

insert()

   |
   v

afterInsert()
```

---

## Réinitialisation

Avant toute opération d'écriture :

```
$this->reset();
```

est appelée.

Cette méthode :

- supprime les anciennes erreurs ;
- remet les valeurs des colonnes à `null`.

Cela garantit qu'une opération précédente ne pollue pas la suivante.

---

## Détermination des colonnes autorisées

La méthode :

```
getInsertColumns()
```

détermine les colonnes pouvant participer à un ajout.

Par défaut :

```
protected function getInsertColumns(): array
```

retourne toutes les colonnes dont la propriété :

```
Insertable
```

est vraie.

Exemple :

```
new ColumnText(
    required: true,
    insertable: true,
    updatable: true
)
```

signifie que la colonne :

- doit être contrôlée lors d'un ajout ;
- peut être insérée en base ;
- pourra également être modifiée.

---

# Propriété `Required`

La propriété :

```
Required
```

indique si une valeur est obligatoire.

Elle intervient uniquement lors de la validation.

Exemple :

```
new ColumnText(
    required: true
)
```

signifie :

- la colonne doit être présente dans les données reçues ;
- une valeur valide doit être fournie.

Exemple :

```
[
    'nom' => 'Dupont'
]
```

est obligatoire si :

```
nom.Required = true
```

---

Une colonne peut être :

```
Required = true
Insertable = false
Updatable = false
```

Exemple :

une valeur calculée automatiquement par l'application.

Elle existe dans le modèle mais n'est pas fournie directement lors d'un ajout.

---

# Propriété `Insertable`

La propriété :

```
Insertable
```

définit si une colonne peut participer à une insertion SQL.

Elle est utilisée par :

```
getInsertColumns()
```

et :

```
insert()
```

---

Exemples courants :

## Clé primaire auto-incrémentée

```
id

Required    = false
Insertable  = false
Updatable   = false
```

La base de données génère elle-même la valeur.

---

## Colonne utilisateur

```
nom

Required    = true
Insertable  = true
Updatable   = true
```

La valeur est fournie lors de l'ajout et modifiable ensuite.

---

## Colonne technique

```
dateCreation

Required    = false
Insertable  = true
Updatable   = false
```

La date peut être enregistrée à la création mais jamais modifiée.

---

# Propriété `Updatable`

La propriété :

```
Updatable
```

définit si une colonne peut être modifiée.

Elle est utilisée par :

```
getUpdateColumns()
```

et :

```
update()
```

---

Exemple :

```
licence

Required    = true
Insertable  = true
Updatable   = false
```

Une licence est attribuée à la création mais ne peut plus changer.

---

# Différence entre Required, Insertable et Updatable

Ces trois propriétés ont des rôles différents.

|Propriété|Rôle|Utilisation|
|---|---|---|
|Required|valeur obligatoire|validation|
|Insertable|autorisée lors d'un ajout|INSERT|
|Updatable|autorisée lors d'une modification|UPDATE|

---

Exemple complet :

```
$this->addColumn(
    'nom',
    new ColumnText(
        required: true,
        insertable: true,
        updatable: true
    )
);
```

La colonne :

- doit être fournie lors d'un ajout ;
- est enregistrée dans l'INSERT ;
- peut être modifiée ensuite.

---

# Validation des données

La méthode :

```
checkColumns()
```

réalise plusieurs contrôles.

Elle vérifie :

- les colonnes interdites ;
- les colonnes obligatoires ;
- les règles propres aux objets `Column`.

---

## Contrôle des colonnes interdites

Les données reçues sont comparées aux colonnes autorisées.

Exemple :

Données reçues :

```
[
    'nom' => 'Dupont',
    'secret' => 'xxx'
]
```

Colonnes autorisées :

```
[
    'nom'
]
```

Résultat :

```
secret :
Cette colonne n'est pas autorisée.
```

---

## Contrôle des colonnes obligatoires

Les colonnes obligatoires sont calculées avec :

```
getRequiredColumns()
```

Cette méthode utilise :

```
Column->Required
```

---

Exemple :

```
nom.Required = true
prenom.Required = true
```

Données :

```
[
    'nom'=>'Dupont'
]
```

Résultat :

```
prenom :
Cette colonne est obligatoire.
```

---

# Validation par les objets Column

Après les contrôles généraux, chaque valeur est confiée à son objet `Column`.

Exemple :

```
$input->checkValidity();
```

La colonne applique alors ses propres règles :

- type ;
- longueur ;
- expression régulière ;
- valeur minimale ;
- valeur maximale ;
- format particulier.

---

# Gestion des champs optionnels

Un champ optionnel vide doit être transmis avec :

```
null
```

et non :

```
''
```

Exemple incorrect :

```
[
    'telephone'=>''
]
```

Exemple correct :

```
[
    'telephone'=>null
]
```

Cette règle permet de distinguer :

- une absence volontaire de valeur ;
- une chaîne vide saisie par erreur.

---

# Contrôle des contraintes d'unicité

Après validation des colonnes :

```
checkUniqueConstraints()
```

est appelée.

Elle vérifie que les données ne créent pas de doublon.

Exemple :

```
nom + prenom + dateNaissance
```

peut constituer une contrainte unique.

---

# Insertion SQL

La méthode interne :

```
protected function insert(): void
```

construit automatiquement la requête :

```
INSERT INTO table
(
    colonne1,
    colonne2
)
VALUES
(
    :colonne1,
    :colonne2
)
```

Seules les colonnes :

- déclarées dans le modèle ;
- autorisées par `Insertable` ;
- possédant une valeur ;

sont intégrées.

---

# Identifiant généré

Après insertion :

```
$this->lastInsertId
```

contient l'identifiant créé.

Accessible par :

```
getLastInsertId()
```

Cela permet aux classes métier d'utiliser l'identifiant nouvellement créé dans :

```
afterInsert()
```

---

La partie suivante détaillera **la modification (`modify`), la suppression (`delete`) et les hooks métier (`beforeInsert`, `beforeUpdate`, `afterInsert`, etc.)**.

soit gentil et comme pour les autres parties donne le tout en markdown

Vous avez raison : je vous ai fourni la partie 4 sous forme de réponse mise en forme, mais pas sous forme de **bloc Markdown brut prêt à copier** comme les parties précédentes.  
Je reprends donc la **partie 4 complète dans un bloc Markdown**.

````
# Gestion des opérations CRUD

La classe `Table` fournit une implémentation générique des opérations de création, modification et suppression.

Les classes métier n'ont donc pas à écrire :

- les requêtes SQL ;
- la gestion des paramètres ;
- les contrôles de présence ;
- la gestion des erreurs techniques ;
- la récupération des identifiants générés.

Elles fournissent uniquement :

- les données reçues ;
- la configuration des colonnes ;
- les règles métier spécifiques.

---

# Ajout d'un enregistrement

La méthode publique :

```php
public function add(array $data): bool
````

permet d'insérer un nouvel enregistrement.

Elle applique automatiquement le cycle suivant :

```
add()

   |
   v

reset()

   |
   v

getInsertColumns()

   |
   v

populateData()

   |
   v

checkColumns()

   |
   v

checkUniqueConstraints()

   |
   v

beforeChange()

   |
   v

beforeInsert()

   |
   v

insert()

   |
   v

afterInsert()
```

---

# Réinitialisation

Avant toute opération d'écriture :

```
$this->reset();
```

est appelée.

Cette méthode :

- supprime les anciennes erreurs ;
- remet les valeurs des colonnes à `null`.

Cela garantit qu'une opération précédente ne pollue pas l'opération suivante.

---

# Détermination des colonnes utilisables lors d'un ajout

La méthode :

```
protected function getInsertColumns(): array
```

détermine les colonnes pouvant participer à une insertion.

Elle utilise la propriété :

```
Column->Insertable
```

Seules les colonnes dont :

```
Insertable = true
```

sont prises en compte.

Exemple :

```
$this->addColumn(
    'nom',
    new ColumnText(
        required: true,
        insertable: true,
        updatable: true
    )
);
```

La colonne `nom` :

- est contrôlée lors d'un ajout ;
- est intégrée dans l'INSERT ;
- pourra être modifiée ensuite.

---

# Les propriétés Required, Insertable et Updatable

Chaque colonne métier possède trois propriétés importantes permettant de définir son comportement dans les opérations CRUD.

Ces propriétés ne remplacent pas les règles de validation propres à `Column`.

Elles indiquent uniquement comment `Table` doit utiliser la colonne.

---

# Propriété Required

La propriété :

```
Required
```

indique si une valeur est obligatoire.

Elle intervient lors de la validation des données.

Exemple :

```
nom.Required = true
```

signifie que la colonne `nom` doit obligatoirement être fournie lors d'un ajout ou d'une modification complète.

Si la donnée est absente :

```
[
    'prenom' => 'Jean'
]
```

alors une erreur est générée :

```
nom :
Cette colonne est obligatoire.
```

---

Une colonne peut être obligatoire sans être directement insérable.

Exemple :

Une date calculée automatiquement :

```
dateCreation

Required    = true
Insertable  = false
Updatable   = false
```

La valeur devra exister dans le modèle mais sera fournie automatiquement par l'application.

---

# Propriété Insertable

La propriété :

```
Insertable
```

indique si la colonne peut être utilisée lors d'une insertion SQL.

Elle intervient dans :

```
getInsertColumns()
```

et :

```
insert()
```

---

Exemples courants.

## Clé primaire auto-incrémentée

```
id

Required    = false
Insertable  = false
Updatable   = false
```

La valeur est générée par la base de données.

---

## Colonne saisie par l'utilisateur

```
nom

Required    = true
Insertable  = true
Updatable   = true
```

La colonne participe aux opérations :

- INSERT ;
- UPDATE.

---

## Colonne technique créée automatiquement

```
dateCreation

Required    = false
Insertable  = true
Updatable   = false
```

Elle est enregistrée lors de la création mais ne peut plus être modifiée.

---

# Propriété Updatable

La propriété :

```
Updatable
```

indique si la colonne peut être modifiée.

Elle intervient dans :

```
getUpdateColumns()
```

et :

```
update()
```

---

Exemple :

```
licence

Required    = true
Insertable  = true
Updatable   = false
```

Une licence peut être créée mais ne peut plus changer ensuite.

---

# Synthèse des trois propriétés

|Propriété|Rôle|Utilisation|
|---|---|---|
|Required|Définit si une valeur est obligatoire|Validation|
|Insertable|Autorise l'utilisation dans un INSERT|Ajout|
|Updatable|Autorise l'utilisation dans un UPDATE|Modification|

---

# Exemple complet de définition d'une colonne

```
$this->addColumn(
    'nom',
    new ColumnText(
        required: true,
        insertable: true,
        updatable: true
    )
);
```

Comportement :

- la colonne doit être fournie ;
- elle est insérée lors d'un ajout ;
- elle est modifiable ensuite.

---

# Validation des données

La méthode :

```
protected function checkColumns(
    array $data,
    array $allowedColumns,
    array $requiredColumns
): bool
```

réalise la validation générique.

Elle contrôle :

- les colonnes interdites ;
- les colonnes obligatoires ;
- les règles propres aux objets `Column`.

---

# Contrôle des colonnes interdites

Les données reçues sont comparées aux colonnes autorisées.

Exemple :

Données :

```
[
    'nom' => 'Dupont',
    'secret' => 'xxx'
]
```

Colonnes autorisées :

```
[
    'nom'
]
```

Résultat :

```
secret :
Cette colonne n'est pas autorisée.
```

---

# Contrôle des colonnes obligatoires

Les colonnes obligatoires sont déterminées par :

```
getRequiredColumns()
```

Cette méthode utilise :

```
Column->Required
```

Exemple :

```
nom.Required = true
prenom.Required = true
```

Données reçues :

```
[
    'nom' => 'Dupont'
]
```

Résultat :

```
prenom :
Cette colonne est obligatoire.
```

---

# Validation par les objets Column

Après les contrôles généraux, chaque valeur est transmise à son objet `Column`.

Exemple :

```
$input->checkValidity();
```

Chaque objet `Column` applique alors ses propres règles :

- type ;
- longueur ;
- expression régulière ;
- valeurs autorisées ;
- bornes minimales ou maximales ;
- format spécifique.

---

# Gestion des champs optionnels

Un champ optionnel vide doit être transmis avec :

```
null
```

et non :

```
''
```

Exemple incorrect :

```
[
    'telephone' => ''
]
```

Exemple correct :

```
[
    'telephone' => null
]
```

Cette règle permet de différencier :

- une absence volontaire de valeur ;
- une chaîne vide issue d'une saisie incorrecte.

---

# Contrôle des contraintes d'unicité

Après validation :

```
checkUniqueConstraints()
```

est exécutée.

Cette méthode vérifie que les données ne créent pas un doublon.

Exemple :

```
nom + prenom + dateNaissance
```

peut constituer une contrainte unique.

Lors d'une modification, l'enregistrement courant est exclu du contrôle.

---

# Insertion SQL automatique

La méthode :

```
protected function insert(): void
```

construit automatiquement la requête SQL.

Exemple généré :

```
INSERT INTO club
(
    nom,
    ville
)
VALUES
(
    :nom,
    :ville
)
```

Seules les colonnes :

- déclarées dans la classe métier ;
- autorisées par `Insertable` ;
- contenant une valeur ;

sont utilisées.

---

# Récupération de l'identifiant créé

Après insertion :

```
$this->lastInsertId
```

contient l'identifiant généré.

Il est accessible par :

```
getLastInsertId()
```

Il peut être utilisé notamment dans :

```
afterInsert()
```

pour effectuer un traitement complémentaire après création.

Modification d'un enregistrement

La méthode publique :

```php
public function modify(mixed $id, array $data): bool
````

permet de modifier un enregistrement existant.

Comme pour l'ajout, la classe `Table` prend en charge :

- la vérification de l'existence de l'enregistrement ;
- la validation des données ;
- le contrôle des contraintes d'unicité ;
- l'appel aux règles métier ;
- la génération de la requête SQL.

La classe métier ne fournit que :

- l'identifiant de l'enregistrement ;
- les valeurs à modifier.

---

# Cycle d'une modification

Le traitement suit le cycle suivant :

```
modify()

    |
    v

reset()

    |
    v

getUpdateColumns()

    |
    v

populateData()

    |
    v

Vérification existence

    |
    v

Chargement des valeurs existantes si nécessaire

    |
    v

checkColumns()

    |
    v

checkUniqueConstraints()

    |
    v

beforeChange()

    |
    v

beforeUpdate()

    |
    v

update()

    |
    v

afterUpdate()
```

---

# Réinitialisation

Comme pour l'ajout :

```
$this->reset();
```

est exécutée au début du traitement.

Cette étape :

- supprime les anciennes erreurs ;
- remet les valeurs des colonnes à zéro.

Chaque opération est donc indépendante.

---

# Détermination des colonnes modifiables

La méthode :

```
protected function getUpdateColumns(): array
```

détermine les colonnes pouvant être modifiées.

Elle utilise :

```
Column->Updatable
```

Une colonne est modifiable uniquement si :

```
Updatable = true
```

---

Exemple :

```
$this->addColumn(
    'nom',
    new ColumnText(
        required: true,
        insertable: true,
        updatable: true
    )
);
```

La colonne `nom` :

- peut être créée ;
- peut être modifiée.

---

Exemple d'une colonne non modifiable :

```
$this->addColumn(
    'dateCreation',
    new ColumnDate(
        required: true,
        insertable: true,
        updatable: false
    )
);
```

La colonne :

- est enregistrée lors de la création ;
- ne peut plus être changée.

---

# Mise à jour complète ou partielle

La méthode `modify()` accepte deux modes de fonctionnement.

## Modification complète

Toutes les colonnes modifiables sont transmises.

Exemple :

```
[
    'nom' => 'Dupont',
    'ville' => 'Paris'
]
```

La validation contrôle alors que toutes les colonnes attendues sont présentes.

---

## Modification partielle

Seules certaines colonnes sont fournies.

Exemple :

```
[
    'ville' => 'Lille'
]
```

Dans ce cas :

- l'enregistrement actuel est chargé depuis la base ;
- les valeurs absentes sont conservées ;
- les contrôles métier disposent de l'ensemble des données.

---

# Chargement des valeurs existantes

Lors d'une modification partielle :

```
$row = $this->getRowById($id);
```

récupère l'enregistrement courant.

Les valeurs sont ensuite replacées dans les objets `Column`.

Cela permet notamment de conserver la cohérence des contrôles :

- contraintes d'unicité ;
- règles métier ;
- calculs dépendant d'autres colonnes.

---

# Vérification de l'existence

Avant une modification complète :

```
$this->exists($id)
```

vérifie que l'enregistrement existe.

Si ce n'est pas le cas :

```
Cet enregistrement n'existe pas.
```

est ajouté aux erreurs.

---

# Validation des données modifiées

La méthode :

```
checkColumns()
```

est ensuite appelée.

Elle vérifie :

- que seules les colonnes autorisées sont utilisées ;
- que les valeurs sont conformes ;
- que les règles des objets `Column` sont respectées.

Les colonnes autorisées sont celles retournées par :

```
getUpdateColumns()
```

---

# Contrôle des contraintes d'unicité lors d'une modification

Lors d'une modification :

```
checkUniqueConstraints($id)
```

est appelée.

L'identifiant courant est exclu de la recherche.

Exemple :

Un club :

```
id = 10
nom = AS Valenciennes
```

est modifié sans changer son nom.

La recherche d'un doublon ne doit pas considérer l'enregistrement lui-même comme un doublon.

---

# Mise à jour SQL automatique

La méthode :

```
protected function update(mixed $id): void
```

génère automatiquement la requête SQL.

Exemple :

```
UPDATE club
SET nom = :nom,
    ville = :ville
WHERE id = :id
```

Seules les colonnes :

- autorisées par `Updatable` ;
- présentes dans la configuration ;
- validées ;

sont utilisées.

---

# Suppression d'un enregistrement

La méthode publique :

```
public function delete(mixed $id): bool
```

supprime un enregistrement.

La suppression suit le cycle :

```
delete()

    |
    v

reset()

    |
    v

Vérification existence

    |
    v

beforeDelete()

    |
    v

DELETE SQL

    |
    v

afterDelete()
```

---

# Vérification avant suppression

Avant toute suppression :

```
$this->exists($id)
```

vérifie que l'enregistrement existe.

Si aucun enregistrement correspondant n'est trouvé :

```
Cet enregistrement n'existe pas.
```

est retourné.

---

# Protection métier avant suppression

La méthode :

```
protected function beforeDelete(mixed $id): bool
```

permet à une classe métier d'interdire une suppression.

Exemple :

Une catégorie ne doit pas être supprimée si des coureurs lui sont associés.

La classe métier peut écrire :

```
protected function beforeDelete(mixed $id): bool
{
    if ($this->hasRunners($id)) {
        $this->addError(
            'global',
            'Cette catégorie contient encore des coureurs.'
        );

        return false;
    }

    return true;
}
```

Retour :

```
true
```

La suppression continue.

Retour :

```
false
```

La suppression est annulée.

---

# Les hooks métier

Les hooks sont des points d'extension permettant aux classes métier d'ajouter leur propre logique.

Ils évitent de modifier la classe générique `Table`.

---

# beforeChange()

Signature :

```
protected function beforeChange(): bool
```

Cette méthode est appelée avant :

- un ajout ;
- une modification.

Elle sert aux règles communes aux deux opérations.

Exemples :

- calculer une valeur ;
- vérifier une cohérence entre plusieurs champs ;
- compléter une donnée.

Exemple :

```
protected function beforeChange(): bool
{
    if ($this->getValue('ageMin') > $this->getValue('ageMax')) {
        $this->addError(
            'global',
            'L’âge minimum doit être inférieur à l’âge maximum.'
        );

        return false;
    }

    return true;
}
```

---

# beforeInsert()

Signature :

```
protected function beforeInsert(): bool
```

Cette méthode est appelée uniquement avant un ajout.

Elle permet :

- de préparer des données spécifiques ;
- d'effectuer un contrôle avant insertion.

Exemple :

```
protected function beforeInsert(): bool
{
    $this->setValue(
        'dateCreation',
        date('Y-m-d')
    );

    return true;
}
```

---

# beforeUpdate()

Signature :

```
protected function beforeUpdate(mixed $id): bool
```

Cette méthode est appelée uniquement avant une modification.

Elle permet :

- de contrôler une modification ;
- d'empêcher certains changements ;
- de compléter les données.

Exemple :

```
protected function beforeUpdate(mixed $id): bool
{
    if ($this->getValue('statut') === 'archive') {
        $this->addError(
            'statut',
            'Un élément archivé ne peut plus être modifié.'
        );

        return false;
    }

    return true;
}
```

---

# afterInsert()

Signature :

```
protected function afterInsert(mixed $id): bool
```

Cette méthode est appelée après une insertion réussie.

Utilisations possibles :

- créer des données liées ;
- enregistrer un historique ;
- effectuer une opération complémentaire.

---

# afterUpdate()

Signature :

```
protected function afterUpdate(mixed $id): bool
```

Cette méthode est appelée après une modification réussie.

Exemples :

- journalisation ;
- synchronisation ;
- recalcul.

---

# afterDelete()

Signature :

```
protected function afterDelete(mixed $id): bool
```

Cette méthode est appelée après une suppression réussie.

Exemples :

- supprimer des fichiers associés ;
- nettoyer des données dépendantes ;
- écrire un historique.

---

# Principe général des hooks

Les hooks permettent de respecter la séparation des responsabilités.

La classe `Table` gère :

- SQL ;
- validation générique ;
- sécurité des opérations.

La classe métier gère :

- règles fonctionnelles ;
- contrôles spécifiques ;
- traitements complémentaires.

Architecture :

```
                 Classe métier

                      |
                      |
                règles spécifiques

                      |
                      v

                    Table

                      |
                      |

          validation + CRUD générique

                      |
                      v

                  Base SQL
```

La classe `Table` reste donc totalement indépendante du métier tout en offrant des points d'extension complets.

# la méthode addServiceError

```
public function addServiceError(string $field, string $message): void
```

Elle existe justement pour permettre aux services d'ajouter des erreurs métier externes.