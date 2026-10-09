# Rôle

`ColumnUrl` est une classe de validation utilisée pour représenter une colonne métier contenant une adresse URL.

Elle hérite de la classe abstraite `Column` et ajoute les contrôles spécifiques aux adresses web :

- présence obligatoire ou non ;
- format général de l'URL ;
- vérification optionnelle de l'accessibilité de la ressource distante.

Elle est utilisée dans les classes métier pour les colonnes SQL contenant :

- un site internet ;
- un lien externe ;
- une ressource accessible par navigateur ;
- une adresse vers un document ou un service en ligne.

# Exemples d'utilisation

Dans une classe métier :

```php
$col = new ColumnUrl(
    required: false
);

$this->addColumn('siteWeb', $col);
````

La colonne `siteWeb` pourra contenir une URL valide ou rester vide.

# Paramètres du constructeur

```
public function __construct(
    bool $required = true,
    bool $insertable = true,
    bool $updatable = true,
    bool $verifierExistence = false
)
```

## required

Indique si l'URL est obligatoire.

Exemple :

```
required: true
```

Une valeur doit obligatoirement être fournie.

Valeurs refusées :

```
null
""
"   "
```

Le contrôle est hérité de la classe `Column`.

## insertable

Indique si la colonne peut être renseignée lors d'une insertion.

Exemple :

```
insertable: false
```

Cas d'utilisation :

- URL calculée automatiquement ;
- lien généré par l'application.

## updatable

Indique si la colonne peut être modifiée.

Exemple :

```
updatable: false
```

Cas d'utilisation :

- lien historique ;
- URL associée définitivement à un élément.

## verifierExistence

Indique si l'application doit vérifier que la ressource distante existe réellement.

Valeur par défaut :

```
false
```

Exemple :

```
new ColumnUrl(
    verifierExistence: true
);
```

Dans ce cas, la classe effectuera une requête vers l'adresse indiquée.

# Fonctionnement de la validation

La méthode :

```
checkValidity()
```

effectue plusieurs contrôles successifs.

# 1 - Vérification de l'obligation

La première étape est héritée de `Column`.

Exemple :

```
$col = new ColumnUrl(
    required: true
);
```

Valeur reçue :

```
null
```

Résultat :

```
Veuillez renseigner ce champ.
```

# 2 - Nettoyage de la valeur

Avant validation, les espaces inutiles sont supprimés.

Exemple :

Valeur reçue :

```
" https://www.exemple.fr "
```

Après traitement :

```
https://www.exemple.fr
```

# 3 - Vérification du format URL

La classe utilise le validateur PHP :

```
FILTER_VALIDATE_URL
```

Exemples acceptés :

```
https://www.exemple.fr
http://www.site.com/page.html
https://api.exemple.com/service?id=10
```

Exemples refusés :

```
www.exemple.fr
exemple.fr
mon site internet
```

Message retourné :

```
L'URL n'est pas valide.
```

# 4 - Vérification optionnelle de l'existence

Lorsque :

```
verifierExistence: true
```

la classe tente de contacter la ressource distante.

Exemple :

```
$col = new ColumnUrl(
    verifierExistence: true
);
```

Valeur :

```
https://www.exemple.fr
```

La classe vérifie que le serveur répond correctement.

Si la ressource est inaccessible :

```
L'URL ne correspond pas à une ressource accessible.
```

# Exemple complet dans une classe métier

```
protected function defineColumns(): void
{
    $col = new ColumnUrl(
        required: false,
        verifierExistence: true
    );

    $this->addColumn('siteWeb', $col);
}
```

Cette déclaration garantit que :

- l'adresse est facultative ;
- si elle est renseignée, son format est valide ;
- le site doit être accessible.

# Exemple avec une table association

Table :

```
association
------------------
id
nom
siteWeb
```

Déclaration :

```
protected function defineColumns(): void
{
    $this->addColumn(
        'siteWeb',
        new ColumnUrl(
            required: false
        )
    );
}
```

Résultats :

|Valeur saisie|Résultat|
|---|---|
|`https://club.fr`|Acceptée|
|`club.fr`|Refusée|
|vide|Acceptée si `required=false`|

# Attention à la vérification distante

La vérification :

```
get_headers($url)
```

implique un accès réseau.

Elle peut échouer dans certains cas :

- serveur distant temporairement indisponible ;
- délai de réponse trop long ;
- restriction réseau ;
- site bloquant les requêtes automatiques.

Cette option doit donc être utilisée uniquement lorsque l'accessibilité immédiate de la ressource est une règle métier.

# Utilisation recommandée

## Cas où vérifier l'existence est pertinent

Exemples :

- lien vers un document obligatoire ;
- URL d'une API externe utilisée immédiatement ;
- ressource devant être disponible au moment de l'enregistrement.

```
new ColumnUrl(
    verifierExistence: true
);
```

## Cas où ne pas vérifier l'existence est préférable

Exemples :

- site personnel d'un utilisateur ;
- lien fourni comme information ;
- URL pouvant être temporairement indisponible.

```
new ColumnUrl(
    verifierExistence: false
);
```

# Résumé

|Fonction|Gestion|
|---|---|
|Valeur obligatoire|Oui|
|Nettoyage des espaces|Oui|
|Contrôle du format URL|Oui|
|Vérification distante optionnelle|Oui|
|Valeur normalisée|Oui|
|Compatible CRUD|Oui|

`ColumnUrl` permet donc de sécuriser les colonnes contenant des liens internet tout en laissant le choix entre un simple contrôle de format et une vérification réelle de disponibilité.