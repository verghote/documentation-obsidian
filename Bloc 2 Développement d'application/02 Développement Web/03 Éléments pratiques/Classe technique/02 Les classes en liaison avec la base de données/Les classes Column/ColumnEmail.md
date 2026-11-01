# Rôle

`ColumnEmail` est une classe de validation utilisée pour représenter une colonne métier contenant une adresse électronique.

Elle hérite de la classe abstraite `Column` et ajoute les contrôles spécifiques aux adresses email :

- présence obligatoire ou non ;
- format général de l'adresse ;
- existence du domaine associé ;
- longueur maximale.

Elle est utilisée dans les classes métier pour les colonnes SQL contenant des informations de contact.

# Exemples d'utilisation

Dans une classe métier :

```php
$col = new ColumnEmail(
    required: true,
    maxLength: 100
);

$this->addColumn('email', $col);
````

La colonne `email` devra contenir une adresse électronique valide.

# Paramètres du constructeur

```
public function __construct(
    bool $required = true,
    bool $insertable = true,
    bool $updatable = true,
    ?int $maxLength = null
)
```

## required

Indique si l'adresse électronique est obligatoire.

Exemple :

```
required: true
```

La valeur doit obligatoirement être renseignée.

Valeurs refusées :

```
null
""
"   "
```

Le contrôle est réalisé par la classe mère `Column`.

## insertable

Indique si la colonne peut être renseignée lors d'une insertion.

Exemple :

```
insertable: false
```

Cas d'utilisation :

- adresse générée automatiquement ;
- adresse provenant d'un service externe.

## updatable

Indique si la colonne peut être modifiée.

Exemple :

```
updatable: false
```

Cas d'utilisation :

- identifiant de contact historique ;
- adresse certifiée qui ne doit plus être changée.

## maxLength

Définit la longueur maximale autorisée.

Exemple :

```
maxLength: 100
```

Une adresse dépassant cette longueur sera refusée.

# Fonctionnement de la validation

La méthode :

```
checkValidity()
```

effectue plusieurs contrôles successifs.

# 1 - Vérification de l'obligation

La première vérification est héritée de `Column`.

Exemple :

```
$col = new ColumnEmail(
    required: true
);
```

Valeur :

```
null
```

Résultat :

```
Veuillez renseigner ce champ.
```

# 2 - Nettoyage de la valeur

Avant les contrôles, la valeur est nettoyée avec :

```
trim()
```

Exemple :

Valeur reçue :

```
" contact@example.com "
```

Après traitement :

```
contact@example.com
```

# 3 - Vérification du format email

La classe utilise le validateur PHP :

```
FILTER_VALIDATE_EMAIL
```

Exemples acceptés :

```
contact@example.com
prenom.nom@entreprise.fr
utilisateur+test@domaine.com
```

Exemples refusés :

```
contact@
@example.com
contact.example.com
```

Message retourné :

```
Le format de l'adresse électronique est invalide.
```

# 4 - Vérification du domaine

Après validation du format, la classe vérifie que le domaine existe réellement.

Exemple :

Adresse :

```
contact@entreprise.fr
```

La classe contrôle l'existence d'un enregistrement DNS de messagerie (`MX`) pour :

```
entreprise.fr
```

Si le domaine n'existe pas :

```
Le domaine de l'adresse électronique n'existe pas.
```

# 5 - Contrôle de longueur

Si une longueur maximale est définie :

```
new ColumnEmail(
    maxLength: 50
);
```

Une adresse dépassant 50 caractères sera refusée.

Message retourné :

```
L'adresse électronique ne doit pas dépasser 50 caractères.
```

# Exemple complet dans une classe métier

```
protected function defineColumns(): void
{
    $col = new ColumnEmail(
        required: true,
        maxLength: 100
    );

    $this->addColumn('email', $col);
}
```

Cette déclaration garantit que :

- l'adresse est obligatoire ;
- son format est valide ;
- son domaine existe ;
- sa longueur est compatible avec la base.

# Exemple de validation

Valeur reçue :

```
$col->Value = " utilisateur@example.com ";
```

Après validation :

```
$col->Value = "utilisateur@example.com";
```

Valeur reçue :

```
$col->Value = "utilisateur@domaine-inexistant-test.fr";
```

Résultat :

```
Le domaine de l'adresse électronique n'existe pas.
```

# Utilisation dans une table utilisateur

Exemple :

```
utilisateur
------------------
id
nom
prenom
email
```

Déclaration :

```
protected function defineColumns(): void
{
    $this->addColumn(
        'email',
        new ColumnEmail(
            required: true,
            maxLength: 150
        )
    );
}
```

La validation est alors centralisée dans la définition métier.

# Remarque sur la vérification DNS

Le contrôle du domaine avec :

```
checkdnsrr()
```

apporte une sécurité supplémentaire mais nécessite que le serveur PHP puisse effectuer des recherches DNS.

Dans certains environnements :

- développement hors connexion ;
- tests automatisés ;
- réseaux filtrés ;

ce contrôle peut nécessiter une adaptation.

# Résumé

|Fonction|Gestion|
|---|---|
|Valeur obligatoire|Oui|
|Nettoyage des espaces|Oui|
|Contrôle du format email|Oui|
|Vérification du domaine DNS|Oui|
|Longueur maximale|Oui|
|Valeur normalisée|Oui|
|Compatible CRUD|Oui|

`ColumnEmail` permet donc de garantir qu'une colonne métier contenant une adresse électronique respecte les règles nécessaires avant son enregistrement en base de données.