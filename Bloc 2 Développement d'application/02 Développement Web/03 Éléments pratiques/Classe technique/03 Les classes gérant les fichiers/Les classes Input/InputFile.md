## Présentation

`InputFile` est une classe spécialisée dans la validation d'un fichier téléversé via un formulaire PHP.

Elle encapsule le tableau `$_FILES` fourni par PHP et permet de contrôler un fichier avant son traitement par l'application.

La classe effectue notamment les vérifications suivantes :

- présence du fichier (si le champ est obligatoire) ;
- contrôle des erreurs de téléversement PHP ;
- existence du fichier temporaire ;
- taille maximale autorisée ;
- extension autorisée ;
- type MIME réel du fichier ;
- préparation et nettoyage du nom du fichier.

Une fois la validation réussie, le nom du fichier est normalisé et disponible via les méthodes héritées de `Input`.

> **Important**
> 
> `InputFile` ne déplace pas le fichier et ne réalise aucun stockage.  
> Son rôle est uniquement de vérifier et préparer le fichier avant son utilisation.

# Création d'un objet

L'objet est construit à partir d'un élément du tableau `$_FILES`.

```php
$inputFile = new InputFile($_FILES['document']);
```

Le tableau transmis doit contenir les clés suivantes :

- `name`
- `tmp_name`
- `error`
- `size`

Dans le cas contraire, une exception est levée.

# Configuration

Avant de lancer la validation, il est possible de définir les règles applicables au fichier.

## Extensions autorisées

```php
$inputFile->setLesExtensions([
    'pdf',
    'docx'
]);
```

Les extensions sont automatiquement converties en minuscules.

## Types MIME autorisés

```php
$inputFile->setLesTypes([
    'application/pdf',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
]);
```

Le type MIME est déterminé à partir du contenu réel du fichier (`finfo`) et non de la valeur envoyée par le navigateur.

## Taille maximale

```php
$inputFile->setMaxSize(5_000_000);
```

La taille est exprimée en octets.

Une valeur de `0` désactive ce contrôle.

## Suppression des accents

Par défaut, les accents sont supprimés.

```php
$inputFile->setSansAccent(true);
```

Pour conserver les caractères accentués :

```php
$inputFile->setSansAccent(false);
```

## Transformation de la casse

Le nom du fichier peut être converti :

```php
$inputFile->setCasse('U'); // Majuscules
```

```php
$inputFile->setCasse('L'); // Minuscules
```

```php
$inputFile->setCasse(''); // Inchangé
```

Les seules valeurs autorisées sont :

|Valeur|Effet|
|---|---|
|`''`|aucune transformation|
|`U`|conversion en majuscules|
|`L`|conversion en minuscules|

Une valeur incorrecte provoque une exception.

# Validation

La validation est déclenchée par :

```php
if ($inputFile->checkValidity()) {
    // Le fichier est valide
}
```

En cas d'échec :

```php
echo $inputFile->getValidationMessage();
```

Le résultat de la validation est également disponible via :

```php
$inputFile->isValide();
```

# Cycle de validation

La méthode `checkValidity()` réalise les contrôles dans l'ordre suivant :

```
Fichier reçu
      ▼
Vérification du téléversement
      ▼
Contrôle de la taille
      ▼
Contrôle de l'extension
      ▼
Contrôle du type MIME
      ▼
Validation spécifique
      ▼
Préparation du nom
      ▼
Validation réussie
```

Le traitement s'arrête dès qu'un contrôle échoue.

# Contrôle du téléversement

La classe vérifie :

- qu'un fichier a bien été envoyé lorsque le champ est obligatoire ;
- qu'aucune erreur PHP n'est survenue (`UPLOAD_ERR_*`) ;
- que le fichier temporaire existe.

Les erreurs de téléversement sont converties en messages explicites.

Exemples :

- taille maximale PHP dépassée ;
- fichier partiellement reçu ;
- dossier temporaire absent ;
- erreur d'écriture sur le disque ;
- interruption par une extension PHP.

# Contrôle de la taille

Si une taille maximale est définie, le fichier est refusé lorsqu'elle est dépassée.

Exemple :

```php
$inputFile->setMaxSize(2_000_000);
```

Message obtenu :

```
La taille du fichier (2458000 octets) dépasse la taille autorisée (2000000 octets).
```

# Contrôle de l'extension

L'extension est extraite du nom du fichier puis comparée à la liste des extensions autorisées.

Exemple :

```php
$inputFile->setLesExtensions(['jpg', 'png']);
```

Un fichier `photo.gif` sera refusé.

# Contrôle du type MIME

Le type MIME est obtenu à partir du contenu réel du fichier grâce à l'extension `fileinfo`.

Exemple :

```php
$inputFile->setLesTypes(['image/jpeg', 'image/png']);
```

Cette vérification évite de se fier uniquement à l'extension du fichier.

# Préparation du nom du fichier

Lorsque tous les contrôles sont validés, le nom est préparé automatiquement.

Les opérations réalisées sont :

1. application éventuelle de la casse ;
2. suppression optionnelle des accents ;
3. suppression des caractères non autorisés ;
4. réduction des espaces multiples ;
5. suppression des noms invalides.

Par exemple :

Nom reçu :

```
Présentation été 2026.pdf
```

Après traitement :

```
Presentation ete 2026.pdf
```

Les caractères autorisés sont :

- lettres ;
- chiffres ;
- espace ;
- point (`.`);
- tiret (`-`) ;
- souligné (`_`).

Un nom vide ou commençant par un point est refusé.

---

# Validation spécifique

La méthode

```php
protected function validationSpecifique(): bool
```

est prévue pour être redéfinie dans une classe dérivée.

Par défaut, elle retourne simplement :

```
return true;
```

Elle permet d'ajouter des contrôles propres à un type de fichier.

Exemples :

- dimensions d'une image ;
- résolution ;

# Méthodes d'accès

## Informations sur le fichier

```php
$inputFile->getFile();
```

Retourne le tableau complet du fichier.


```php
$inputFile->getName();
```

Retourne le nom d'origine.


```php
$inputFile->getTmpName();
```

Retourne le chemin du fichier temporaire.


```php
$inputFile->getSize();
```

Retourne la taille en octets.


```php
$inputFile->getError();
```

Retourne le code d'erreur PHP.


```php
$inputFile->getExtension();
```

Retourne l'extension en minuscules.


```php
$inputFile->getMimeType();
```

Retourne le type MIME détecté.


# Messages de validation

Lorsque la validation échoue, le message est accessible via :

```php
echo $inputFile->getValidationMessage();
```

Les principales causes d'échec sont :

- aucun fichier transmis ;
- erreur de téléversement ;
- fichier temporaire introuvable ;
- taille dépassée ;
- extension interdite ;
- type MIME interdit ;
- nom de fichier invalide ;
- validation spécifique échouée.

---

# Exemple complet

```php
$inputFile = new InputFile($_FILES['photo']);

$inputFile->setLesExtensions(['jpg', 'png']);

$inputFile->setLesTypes(['image/jpeg', 'image/png']);

$inputFile->setMaxSize(2_000_000);

$inputFile->setSansAccent(true);

$inputFile->setCasse('');

if (!$inputFile->checkValidity()) {
    echo $inputFile->getValidationMessage();
    exit;
}

$nom = $inputFile->getValue();
$tmp = $inputFile->getTmpName();

// Le fichier est prêt à être transmis à une classe de stockage.
```


# Extension de la classe

`InputFile` est conçue pour être spécialisée.

Une classe dérivée peut conserver tous les contrôles génériques tout en ajoutant ses propres règles.

Exemple :

```
class InputImage extends InputFile
{
    protected function validationSpecifique(): bool
    {
        // Contrôle des dimensions, du format, etc.

        return true;
    }
}
```

Cette approche permet de mutualiser toute la logique de validation des fichiers tout en adaptant les contrôles aux différents types de contenus.

# Résumé

`InputFile` constitue le point d'entrée de la validation des fichiers téléversés.

Elle permet de :

- contrôler le téléversement PHP ;
- vérifier la taille du fichier ;
- vérifier son extension ;
- contrôler son type MIME réel ;
- préparer un nom de fichier exploitable ;
- fournir des messages d'erreur explicites ;
- être facilement spécialisée par héritage.

Le déplacement, le stockage et la suppression des fichiers relèvent d'une classe dédiée de gestion du système de fichiers.