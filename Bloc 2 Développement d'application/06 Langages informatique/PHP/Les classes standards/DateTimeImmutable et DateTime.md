# 1. Introduction

PHP fournit plusieurs classes pour manipuler les dates et les heures.

Les deux classes principales sont :

```php
DateTime
DateTimeImmutable
```

Elles représentent toutes les deux une **date + une heure + un fuseau horaire éventuel**.

Par exemple :

```php
$date = new DateTimeImmutable('2026-09-30 14:30:00');
```

représente :

```text
30 septembre 2026 à 14:30:00
```

PHP fournit également :

- `DateTimeZone` → représente un fuseau horaire
    
- `DateInterval` → représente une durée/intervallle
    
- `DatePeriod` → permet notamment d'itérer sur une période
    
- `DateTimeInterface` → interface commune à `DateTime` et `DateTimeImmutable`
    

`DateTimeInterface` permet notamment d'écrire une méthode acceptant indifféremment les deux types. PPHP

Exemple :

```php
function afficherDate(DateTimeInterface $date): string
{
    return $date->format('d/m/Y');
}
```

Cette fonction accepte :

```php
$date1 = new DateTime();
$date2 = new DateTimeImmutable();

afficherDate($date1);
afficherDate($date2);
```

---

# 2. Comprendre les objets date en PHP

Il faut abandonner l'idée que les dates sont simplement des chaînes.

Ceci :

```php
$date = '2026-09-30';
```

est une chaîne de caractères.

Alors que ceci :

```php
$date = new DateTimeImmutable('2026-09-30');
```

est un **objet spécialisé dans la manipulation des dates**.

Cela permet de faire :

```php
$date->format('d/m/Y');
```

```php
$date->modify('+7 days');
```

```php
$date->diff($autreDate);
```

```php
$date->setTimezone($timezone);
```

etc.

Le système date/heure de PHP gère notamment les fuseaux horaires et les transitions liées à l'heure d'été/hiver. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

# 3. `DateTimeImmutable`

## Définition

`DateTimeImmutable` représente une date et une heure **immuables**.

Le mot important est :

> immutable = qui ne peut pas être modifié.

Cela ne signifie pas qu'on ne peut pas faire évoluer une date.

Cela signifie que les méthodes de modification **retournent un nouvel objet** au lieu de modifier l'objet original.

Exemple :

```php
$date = new DateTimeImmutable('2026-09-30');

$demain = $date->modify('+1 day');
```

On obtient :

```text
$date    = 30/09/2026
$demain  = 01/10/2026
```

L'objet `$date` n'a pas changé.

---

# 4. Créer une date

## Date actuelle

```php
$date = new DateTimeImmutable();
```

Cela signifie :

> maintenant

On peut également écrire :

```php
$date = new DateTimeImmutable('now');
```

---

## Aujourd'hui à minuit

C'est exactement ce que fait :

```php
$aujourdHui = new DateTimeImmutable('today');
```

Par exemple :

```text
30/09/2026 00:00:00
```

---

## Une date précise

```php
$date = new DateTimeImmutable('2026-09-30');
```

---

## Une date avec une heure

```php
$date = new DateTimeImmutable('2026-09-30 14:30:00');
```

---

## Une date au format ISO

```php
$date = new DateTimeImmutable('2026-09-30T14:30:00');
```

---

## Une date avec fuseau horaire

```php
$date = new DateTimeImmutable(
    '2026-09-30 14:30:00',
    new DateTimeZone('Europe/Paris')
);
```

---

## Expressions relatives

PHP comprend de nombreuses expressions :

```php
new DateTimeImmutable('tomorrow');
```

```php
new DateTimeImmutable('yesterday');
```

```php
new DateTimeImmutable('next monday');
```

```php
new DateTimeImmutable('last friday');
```

```php
new DateTimeImmutable('+3 days');
```

```php
new DateTimeImmutable('-2 weeks');
```

```php
new DateTimeImmutable('+1 month');
```

Exemple :

```php
$lundi = new DateTimeImmutable('next monday');

echo $lundi->format('d/m/Y');
```

Le constructeur accepte une chaîne représentant une date/heure et un `DateTimeZone` optionnel. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

# 5. Formater une date avec `format()`

Une date n'est généralement pas affichée directement.

On utilise :

```php
$date->format(...)
```

Exemple :

```php
$date = new DateTimeImmutable('2026-09-30');

echo $date->format('d/m/Y');
```

Résultat :

```text
30/09/2026
```

---

## Exemples

```php
$date->format('Y');
```

```text
2026
```

```php
$date->format('m');
```

```text
09
```

```php
$date->format('d');
```

```text
30
```

```php
$date->format('H');
```

```text
14
```

```php
$date->format('i');
```

```text
30
```

```php
$date->format('s');
```

```text
00
```

---

# 6. Les caractères de format

Voici les principaux caractères à connaître.

## Année

|Format|Signification|Exemple|
|---|---|---|
|`Y`|année sur 4 chiffres|`2026`|
|`y`|année sur 2 chiffres|`26`|

```php
$date->format('Y');
```

---

## Mois

|Format|Signification|Exemple|
|---|---|---|
|`m`|mois avec zéro|`09`|
|`n`|mois sans zéro|`9`|
|`F`|nom complet|`September`|
|`M`|nom abrégé|`Sep`|

---

## Jour

|Format|Signification|Exemple|
|---|---|---|
|`d`|jour avec zéro|`03`|
|`j`|jour sans zéro|`3`|
|`l`|jour complet|`Wednesday`|
|`D`|jour abrégé|`Wed`|

---

## Heure

|Format|Signification|
|---|---|
|`H`|heure 24h|
|`h`|heure 12h|
|`i`|minutes|
|`s`|secondes|
|`v`|millisecondes|
|`u`|microsecondes|
|`a`|am/pm|
|`A`|AM/PM|

Exemple :

```php
echo $date->format('H:i:s');
```

Résultat :

```text
14:30:00
```

---

## Exemple complet

```php
$date = new DateTimeImmutable('2026-09-30 14:35:42');

echo $date->format('d/m/Y H:i:s');
```

Résultat :

```text
30/09/2026 14:35:42
```

---

## Format français classique

```php
$date->format('d/m/Y');
```

Résultat :

```text
30/09/2026
```

---

## Format SQL

```php
$date->format('Y-m-d H:i:s');
```

Résultat :

```text
2026-09-30 14:35:42
```

---

## Format HTML `datetime-local`

```php
$date->format('Y-m-d\TH:i');
```

Résultat :

```text
2026-09-30T14:35
```

Le `\T` est nécessaire pour obtenir littéralement la lettre `T`.

---

## Format ISO 8601

```php
$date->format(DateTimeInterface::ATOM);
```

Résultat similaire à :

```text
2026-09-30T14:35:42+02:00
```

---

# 7. Modifier une date avec `modify()`

`modify()` est probablement l'une des méthodes les plus importantes.

```php
$date = new DateTimeImmutable('2026-09-30');

$nouvelleDate = $date->modify('+1 day');
```

Résultat :

```text
$date          → 30/09/2026
$nouvelleDate  → 01/10/2026
```

L'objet initial n'est pas modifié.

---

## Ajouter un jour

```php
$demain = $date->modify('+1 day');
```

## Retirer un jour

```php
$hier = $date->modify('-1 day');
```

## Ajouter une semaine

```php
$date->modify('+1 week');
```

## Ajouter un mois

```php
$date->modify('+1 month');
```

## Ajouter une année

```php
$date->modify('+1 year');
```

## Ajouter plusieurs jours

```php
$date->modify('+15 days');
```

## Retirer plusieurs mois

```php
$date->modify('-3 months');
```

---

## Expressions plus complexes

```php
$date->modify('next monday');
```

```php
$date->modify('last day of this month');
```

```php
$date->modify('first day of next month');
```

```php
$date->modify('first day of January');
```

---

# 8. `add()` et `sub()`

Une autre façon de modifier une date consiste à utiliser `DateInterval`.

## Ajouter

```php
$interval = new DateInterval('P10D');

$date2 = $date->add($interval);
```

`P10D` signifie :

```text
P = période
10D = 10 jours
```

---

## Soustraire

```php
$date2 = $date->sub($interval);
```

---

## Exemple

```php
$date = new DateTimeImmutable('2026-09-30');

$interval = new DateInterval('P10D');

$dateFuture = $date->add($interval);

echo $date->format('d/m/Y');
echo $dateFuture->format('d/m/Y');
```

Résultat :

```text
30/09/2026
10/10/2026
```

Encore une fois, `$date` reste inchangée.

---

# 9. `DateInterval`

`DateInterval` représente une durée.

## 10 jours

```php
new DateInterval('P10D');
```

## 2 semaines

```php
new DateInterval('P2W');
```

## 3 mois

```php
new DateInterval('P3M');
```

## 1 année

```php
new DateInterval('P1Y');
```

## 1 année, 2 mois et 10 jours

```php
new DateInterval('P1Y2M10D');
```

---

## Avec une durée

Le préfixe `PT` est utilisé lorsqu'on manipule notamment les heures :

```php
new DateInterval('PT2H');
```

= 2 heures.

```php
new DateInterval('PT30M');
```

= 30 minutes.

```php
new DateInterval('PT45S');
```

= 45 secondes.

On peut combiner :

```php
new DateInterval('P1DT2H30M');
```

= 1 jour, 2 heures et 30 minutes.

---

# 10. `diff()`

`diff()` permet de calculer la différence entre deux dates.

```php
$date1 = new DateTimeImmutable('2026-09-01');
$date2 = new DateTimeImmutable('2026-09-30');

$diff = $date1->diff($date2);
```

On peut ensuite récupérer :

```php
$diff->days
```

```php
$diff->d
```

```php
$diff->m
```

```php
$diff->y
```

---

## Exemple

```php
$date1 = new DateTimeImmutable('2026-09-01');
$date2 = new DateTimeImmutable('2026-09-30');

$diff = $date1->diff($date2);

echo $diff->days;
```

Résultat :

```text
29
```

---

## Afficher une différence

```php
echo $diff->format('%a jours');
```

Résultat :

```text
29 jours
```

---

## Les principaux formats de `DateInterval`

|Format|Signification|
|---|---|
|`%y`|années|
|`%m`|mois|
|`%d`|jours|
|`%h`|heures|
|`%i`|minutes|
|`%s`|secondes|
|`%a`|nombre total de jours|
|`%r`|signe `-` si négatif|

Exemple :

```php
echo $diff->format('%m mois et %d jours');
```

---

# 11. Définir une date précise avec `setDate()`

Avec `DateTimeImmutable` :

```php
$date = new DateTimeImmutable();

$date2 = $date->setDate(
    2026,
    12,
    25
);
```

On obtient :

```text
25/12/2026
```

L'objet original reste inchangé.

---

## Exemple

```php
$date = new DateTimeImmutable('2026-09-30');

$noel = $date->setDate(2026, 12, 25);
```

---

# 12. Définir une heure avec `setTime()`

```php
$date = new DateTimeImmutable('2026-09-30');

$date2 = $date->setTime(14, 30);
```

Résultat :

```text
30/09/2026 14:30:00
```

Avec les secondes :

```php
$date2 = $date->setTime(14, 30, 45);
```

---

# 13. Timestamp Unix

Un timestamp Unix représente le nombre de secondes écoulées depuis :

```text
1970-01-01 00:00:00 UTC
```

---

## Obtenir un timestamp

```php
$date = new DateTimeImmutable();

$timestamp = $date->getTimestamp();
```

---

## Créer une date depuis un timestamp

```php
$date = DateTimeImmutable::createFromTimestamp(1759233600);
```

---

## Exemple

```php
$date = new DateTimeImmutable('2026-09-30');

echo $date->getTimestamp();
```

---

## Pourquoi utiliser les timestamps ?

Ils sont pratiques pour :

- comparer des instants ;
    
- stocker une date ;
    
- communiquer avec certaines API ;
    
- mesurer des durées ;
    
- effectuer certaines opérations techniques.
    

Mais pour manipuler des dates métier, `DateTimeImmutable` est généralement beaucoup plus lisible.

---

# 14. Fuseaux horaires avec `DateTimeZone`

Un objet `DateTimeZone` représente un fuseau horaire.

```php
$timezone = new DateTimeZone('Europe/Paris');
```

Puis :

```php
$date = new DateTimeImmutable(
    '2026-09-30 14:00:00',
    $timezone
);
```

---

## Exemples de fuseaux

```php
Europe/Paris
```

```php
Europe/London
```

```php
America/New_York
```

```php
America/Los_Angeles
```

```php
Asia/Tokyo
```

```php
UTC
```

---

# 15. Changer de fuseau horaire

Utiliser :

```php
setTimezone()
```

Avec `DateTimeImmutable` :

```php
$dateParis = new DateTimeImmutable(
    '2026-09-30 14:00:00',
    new DateTimeZone('Europe/Paris')
);

$dateTokyo = $dateParis->setTimezone(
    new DateTimeZone('Asia/Tokyo')
);
```

Important :

> `setTimezone()` change la représentation de l'instant dans un autre fuseau ; ce n'est pas simplement un changement du texte de l'heure.

Par exemple :

```text
Paris : 14:00
Tokyo : 21:00
```

peuvent représenter exactement le même instant.

---

## Récupérer le fuseau

```php
$timezone = $date->getTimezone();
```

---

## Récupérer le décalage

```php
$offset = $date->getOffset();
```

Le résultat est exprimé en secondes.

Par exemple :

```text
7200
```

correspond à :

```text
UTC+02:00
```

---

# 16. Parser une date avec `createFromFormat()`

C'est une méthode extrêmement utile lorsque tu reçois une date provenant :

- d'un formulaire ;
    
- d'une API ;
    
- d'une base de données ;
    
- d'un fichier ;
    
- d'un utilisateur.
    

Supposons que tu reçoives :

```text
30/09/2026
```

Tu peux faire :

```php
$date = DateTimeImmutable::createFromFormat(
    'd/m/Y',
    '30/09/2026'
);
```

Puis :

```php
echo $date->format('Y-m-d');
```

Résultat :

```text
2026-09-30
```

---

## Exemple avec une heure

```php
$date = DateTimeImmutable::createFromFormat(
    'd/m/Y H:i',
    '30/09/2026 14:30'
);
```

---

## Formats SQL

```php
$date = DateTimeImmutable::createFromFormat(
    'Y-m-d H:i:s',
    '2026-09-30 14:30:00'
);
```

---

## Vérifier les erreurs

`createFromFormat()` peut retourner `false`.

Il faut donc être prudent :

```php
$date = DateTimeImmutable::createFromFormat(
    'd/m/Y',
    $input
);

if ($date === false) {
    // Date invalide
}
```

On peut également utiliser :

```php
$errors = DateTimeImmutable::getLastErrors();
```

---

# 17. ISO 8601 et constantes prédéfinies

PHP fournit plusieurs constantes utiles via `DateTimeInterface`.

Par exemple :

```php
DateTimeInterface::ATOM
```

correspond à :

```text
Y-m-d\TH:i:sP
```

---

## Exemple

```php
$date = new DateTimeImmutable();

echo $date->format(DateTimeInterface::ATOM);
```

Résultat :

```text
2026-09-30T14:30:00+02:00
```

---

## Autres constantes

```php
DateTimeInterface::ATOM
DateTimeInterface::COOKIE
DateTimeInterface::ISO8601
DateTimeInterface::ISO8601_EXPANDED
DateTimeInterface::RFC822
DateTimeInterface::RFC850
DateTimeInterface::RFC1036
DateTimeInterface::RFC1123
DateTimeInterface::RFC7231
DateTimeInterface::RFC2822
DateTimeInterface::RFC3339
DateTimeInterface::RFC3339_EXTENDED
DateTimeInterface::RSS
DateTimeInterface::W3C
```

Ces constantes sont communes aux deux classes. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

# 18. `DateTime`

`DateTime` possède pratiquement les mêmes fonctionnalités que `DateTimeImmutable`.

La différence essentielle est la **mutabilité**.

La documentation PHP indique que `DateTime` modifie directement l'objet lorsqu'on utilise des méthodes telles que `modify()`, `add()` ou `sub()`. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

## Création

```php
$date = new DateTime();
```

```php
$date = new DateTime('2026-09-30');
```

```php
$date = new DateTime('2026-09-30 14:30:00');
```

---

## Modification

```php
$date = new DateTime('2026-09-30');

$date->modify('+1 day');
```

Ici, `$date` a été modifiée.

Elle contient maintenant :

```text
01/10/2026
```

---

# 19. La différence fondamentale entre `DateTime` et `DateTimeImmutable`

C'est LE point à comprendre.

## `DateTimeImmutable`

```php
$date = new DateTimeImmutable('2026-09-30');

$demain = $date->modify('+1 day');
```

Résultat :

```text
$date    = 30/09/2026
$demain  = 01/10/2026
```

---

## `DateTime`

```php
$date = new DateTime('2026-09-30');

$demain = $date->modify('+1 day');
```

Résultat :

```text
$date    = 01/10/2026
$demain  = 01/10/2026
```

Les deux variables font référence à l'objet modifié.

---

## Visualisation

### `DateTimeImmutable`

```text
$date
  │
  ▼
30/09/2026

modify('+1 day')

$date ───────────────► 30/09/2026

$demain ─────────────► 01/10/2026
```

### `DateTime`

```text
$date
  │
  ▼
30/09/2026

modify('+1 day')

$date ───────────────► 01/10/2026

$demain ─────────────► 01/10/2026
```

---

# 20. Piège classique avec `DateTime`

Prenons :

```php
$date = new DateTime('2026-09-30');

$debut = $date;

$fin = $date->modify('+10 days');
```

On pourrait penser :

```text
$date  = 30/09/2026
$debut = 30/09/2026
$fin   = 10/10/2026
```

Mais ce n'est pas ce qui se produit.

Avec `DateTime`, `$date`, `$debut` et `$fin` peuvent référencer le même objet.

Après modification :

```text
$date  = 10/10/2026
$debut = 10/10/2026
$fin   = 10/10/2026
```

---

## Pour éviter cela avec `DateTime`

Utiliser `clone` :

```php
$date = new DateTime('2026-09-30');

$debut = clone $date;

$fin = clone $date;
$fin->modify('+10 days');
```

Maintenant :

```text
$debut = 30/09/2026
$fin   = 10/10/2026
```

La documentation PHP recommande justement `DateTimeImmutable` lorsqu'on souhaite éviter ce comportement mutable. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

# 21. Quand utiliser `DateTimeImmutable` ?

Dans la plupart des applications modernes, `DateTimeImmutable` est un excellent choix par défaut.

Il est particulièrement intéressant pour :

- les applications métier ;
    
- les dates de facturation ;
    
- les dates de naissance ;
    
- les échéances ;
    
- les périodes ;
    
- les dates stockées dans des objets ;
    
- les paramètres de fonctions ;
    
- les applications utilisant beaucoup de calculs de dates.
    

Exemple :

```php
function getDateExpiration(
    DateTimeImmutable $dateDebut
): DateTimeImmutable {
    return $dateDebut->modify('+30 days');
}
```

La fonction ne modifie pas accidentellement la date reçue.

---

# 22. Comparaisons de dates

Les objets `DateTime` et `DateTimeImmutable` peuvent être comparés.

```php
$date1 = new DateTimeImmutable('2026-09-01');
$date2 = new DateTimeImmutable('2026-09-30');

if ($date1 < $date2) {
    echo 'date1 est avant date2';
}
```

---

## Égalité

```php
if ($date1 == $date2) {
    // même date/heure
}
```

## Avant

```php
if ($date1 < $date2) {
}
```

## Après

```php
if ($date1 > $date2) {
}
```

---

## Exemple concret

```php
$expiration = new DateTimeImmutable('2026-10-15');
$aujourdHui = new DateTimeImmutable('today');

if ($aujourdHui > $expiration) {
    echo 'Expiré';
} else {
    echo 'Encore valide';
}
```

---

# 23. Exemples concrets

## Obtenir aujourd'hui

```php
$aujourdHui = new DateTimeImmutable('today');

echo $aujourdHui->format('d/m/Y');
```

---

## Obtenir demain

```php
$demain = new DateTimeImmutable('tomorrow');
```

ou :

```php
$demain = new DateTimeImmutable('today')
    ->modify('+1 day');
```

---

## Obtenir hier

```php
$hier = new DateTimeImmutable('yesterday');
```

---

## Dans 30 jours

```php
$date = new DateTimeImmutable();

$dateDans30Jours = $date->modify('+30 days');
```

---

## Il y a 6 mois

```php
$date = new DateTimeImmutable();

$dateIlYASixMois = $date->modify('-6 months');
```

---

# 24. Exemple : année scolaire

Ton exemple :

```php
$aujourdHui = new DateTimeImmutable('today');

$debutAnnee = new DateTimeImmutable(
    $aujourdHui->format('Y') . '-09-01'
);
```

signifie :

1. obtenir aujourd'hui ;
    
2. récupérer l'année courante ;
    
3. construire le 1er septembre de cette année.
    

```php
echo $date->format('Y-m-d');
```

Résultat :

```text
2026-09-30
```

---

## Exemple avec une heure

```php
$date = DateTimeImmutable::createFromFormat(
    'd/m/Y H:i',
    '30/09/2026 14:30'
);
```

---

## Formats SQL

---

## Vérifier les erreurs

Il faut donc être prudent :

```php
$date = DateTimeImmutable::createFromFormat(
    'd/m/Y',
    $input
);

if ($date === false) {
    // Date invalide
}
```

On peut également utiliser :

```php
$errors = DateTimeImmutable::getLastErrors();
```

---

# 17. ISO 8601 et constantes prédéfinies

PHP fournit plusieurs constantes utiles via `DateTimeInterface`.

Par exemple :

```php
DateTimeInterface::ATOM
```

correspond à :

```text
Y-m-d\TH:i:sP
```

---

## Exemple

```php
$date = new DateTimeImmutable();

echo $date->format(DateTimeInterface::ATOM);
```

Résultat :

```text
2026-09-30T14:30:00+02:00
```

---

## Autres constantes

```php
DateTimeInterface::ATOM
DateTimeInterface::COOKIE
DateTimeInterface::ISO8601
DateTimeInterface::ISO8601_EXPANDED
DateTimeInterface::RFC822
DateTimeInterface::RFC850
DateTimeInterface::RFC1036
DateTimeInterface::RFC1123
DateTimeInterface::RFC7231
DateTimeInterface::RFC2822
DateTimeInterface::RFC3339
DateTimeInterface::RFC3339_EXTENDED
DateTimeInterface::RSS
DateTimeInterface::W3C
```

Ces constantes sont communes aux deux classes. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

# 18. `DateTime`

`DateTime` possède pratiquement les mêmes fonctionnalités que `DateTimeImmutable`.

La différence essentielle est la **mutabilité**.

La documentation PHP indique que `DateTime` modifie directement l'objet lorsqu'on utilise des méthodes telles que `modify()`, `add()` ou `sub()`. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

## Création

```php
$date = new DateTime();
```

```php
$date = new DateTime('2026-09-30');
```

```php
$date = new DateTime('2026-09-30 14:30:00');
```

---

## Modification

```php
$date = new DateTime('2026-09-30');

$date->modify('+1 day');
```

Ici, `$date` a été modifiée.

Elle contient maintenant :

```text
01/10/2026
```

---

# 19. La différence fondamentale entre `DateTime` et `DateTimeImmutable`

C'est LE point à comprendre.

## `DateTimeImmutable`

```php
$date = new DateTimeImmutable('2026-09-30');

$demain = $date->modify('+1 day');
```

Résultat :

```text
$date    = 30/09/2026
$demain  = 01/10/2026
```

---

## `DateTime`

```php
$date = new DateTime('2026-09-30');

$demain = $date->modify('+1 day');
```

Résultat :

```text
$date    = 01/10/2026
$demain  = 01/10/2026
```

Les deux variables font référence à l'objet modifié.

---

## Visualisation

### `DateTimeImmutable`

```text
$date
  │
  ▼
30/09/2026

modify('+1 day')

$date ───────────────► 30/09/2026

$demain ─────────────► 01/10/2026
```

### `DateTime`

```text
$date
  │
  ▼
30/09/2026

modify('+1 day')

$date ───────────────► 01/10/2026

$demain ─────────────► 01/10/2026
```

---

# 20. Piège classique avec `DateTime`

Prenons :

```php
$date = new DateTime('2026-09-30');

$debut = $date;

$fin = $date->modify('+10 days');
```

On pourrait penser :

```text
$date  = 30/09/2026
$debut = 30/09/2026
$fin   = 10/10/2026
```

Mais ce n'est pas ce qui se produit.

Avec `DateTime`, `$date`, `$debut` et `$fin` peuvent référencer le même objet.

Après modification :

```text
$date  = 10/10/2026
$debut = 10/10/2026
$fin   = 10/10/2026
```

---

## Pour éviter cela avec `DateTime`

Utiliser `clone` :

```php
$date = new DateTime('2026-09-30');

$debut = clone $date;

$fin = clone $date;
$fin->modify('+10 days');
```

Maintenant :

```text
$debut = 30/09/2026
$fin   = 10/10/2026
```

La documentation PHP recommande justement `DateTimeImmutable` lorsqu'on souhaite éviter ce comportement mutable.

---

# 21. Quand utiliser `DateTimeImmutable` ?

Dans la plupart des applications modernes, `DateTimeImmutable` est un excellent choix par défaut.

Il est particulièrement intéressant pour :

Par exemple, si nous sommes en :

```text
30/09/2026
```

alors :

```php
$aujourdHui->format('Y')
```

donne :

```text
2026
```

et :

```php
$debutAnnee
```

devient :

```text
01/09/2026
```

---

## Attention à un détail métier

Si tu cherches réellement **le début de l'année scolaire en cours**, ton code n'est pas suffisant.

Supposons :

```text
15/03/2026
```

Ton code produira :

```text
01/09/2026
```

qui est dans le futur.

Pour une année scolaire débutant en septembre, il faudrait plutôt :

```php
$aujourdHui = new DateTimeImmutable('today');

if ((int) $aujourdHui->format('n') < 9) {
    $debutAnnee = new DateTimeImmutable(
        ((int) $aujourdHui->format('Y') - 1) . '-09-01'
    );
} else {
    $debutAnnee = new DateTimeImmutable(
        $aujourdHui->format('Y') . '-09-01'
    );
}
```

Ainsi :

```text
15/03/2026 → 01/09/2025
30/09/2026 → 01/09/2026
```

Une version plus compacte :

```php
$aujourdHui = new DateTimeImmutable('today');

$debutAnnee = new DateTimeImmutable(
    (
        (int) $aujourdHui->format('n') < 9
            ? (int) $aujourdHui->format('Y') - 1
            : (int) $aujourdHui->format('Y')
    ) . '-09-01'
);
```

---

# 25. Exemple : début et fin du mois

## Premier jour du mois

```php
$date = new DateTimeImmutable();

$debutMois = $date->modify('first day of this month');
```

## Dernier jour du mois

```php
$finMois = $date->modify('last day of this month');
```

---

## Exemple complet

```php
$date = new DateTimeImmutable('2026-09-15');

$debutMois = $date->modify('first day of this month');
$finMois = $date->modify('last day of this month');

echo $debutMois->format('d/m/Y');
echo $finMois->format('d/m/Y');
```

Résultat :

```text
01/09/2026
30/09/2026
```

---

# 26. Exemple : début et fin d'année

## Début de l'année

```php
$debutAnnee = $date->modify('first day of January');
```

Ou :

```php
$debutAnnee = $date->setDate(
    (int) $date->format('Y'),
    1,
    1
);
```

---

## Fin de l'année

```php
$finAnnee = $date->modify('last day of December');
```

---

# 27. Exemple : dates relatives

PHP permet d'utiliser des expressions très pratiques.

```php
$date = new DateTimeImmutable('next monday');
```

```php
$date = new DateTimeImmutable('last monday');
```

```php
$date = new DateTimeImmutable('first day of next month');
```

```php
$date = new DateTimeImmutable('last day of this month');
```

```php
$date = new DateTimeImmutable('first day of January 2027');
```

---

## Exemple : prochain lundi

```php
$lundi = new DateTimeImmutable('next monday');

echo $lundi->format('d/m/Y');
```

---

# 28. Exemple : vérifier si une date est passée

```php
$expiration = new DateTimeImmutable('2026-12-31');
$aujourdHui = new DateTimeImmutable('today');

if ($expiration < $aujourdHui) {
    echo 'La date est dépassée';
}
```

---

## Vérifier si une date est future

```php
if ($expiration > $aujourdHui) {
    echo 'La date est dans le futur';
}
```

---

# 29. Exemple : âge

Supposons :

```php
$naissance = new DateTimeImmutable('1990-05-15');
$aujourdHui = new DateTimeImmutable('today');

$age = $naissance->diff($aujourdHui)->y;

echo $age;
```

La propriété :

```php
->y
```

contient le nombre d'années complètes de l'intervalle.

---

# 30. Exemple : délai restant

Supposons :

```php
$maintenant = new DateTimeImmutable();

$evenement = new DateTimeImmutable('2026-12-25 18:00:00');

$difference = $maintenant->diff($evenement);

echo $difference->format(
    '%a jours, %h heures et %i minutes'
);
```

---

# 31. Exemple : liste de dates

Grâce à l'immutabilité, on peut facilement construire plusieurs dates à partir d'une même date de référence.

```php
$date = new DateTimeImmutable('2026-09-01');

$debut = $date;
$semaineSuivante = $date->modify('+1 week');
$moisSuivant = $date->modify('+1 month');
$anneeSuivante = $date->modify('+1 year');
```

On obtient :

```text
$debut          01/09/2026
$semaineSuivante 08/09/2026
$moisSuivant     01/10/2026
$anneeSuivante   01/09/2027
```

Et surtout :

```php
$date
```

reste :

```text
01/09/2026
```

C'est une des grandes forces de `DateTimeImmutable`.

---

# 32. Conversion `DateTime` ↔ `DateTimeImmutable`

Il peut arriver qu'une bibliothèque fournisse un `DateTime` alors que ton application utilise `DateTimeImmutable`.

## `DateTime` vers `DateTimeImmutable`

```php
$dateImmutable = DateTimeImmutable::createFromMutable(
    $dateMutable
);
```

---

## `DateTimeImmutable` vers `DateTime`

```php
$dateMutable = DateTime::createFromImmutable(
    $dateImmutable
);
```

---

## Depuis `DateTimeInterface`

`DateTimeImmutable` possède également :

```php
DateTimeImmutable::createFromInterface($date)
```

Cela permet de créer une instance immutable à partir d'un objet implémentant `DateTimeInterface`. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

# 33. Méthodes principales — tableau récapitulatif

## `DateTimeImmutable`

|Méthode|Rôle|Modifie l'objet ?|
|---|---|---|
|`format()`|formater|Non|
|`modify()`|modifier une date|Non|
|`add()`|ajouter un intervalle|Non|
|`sub()`|soustraire un intervalle|Non|
|`diff()`|calculer une différence|Non|
|`setDate()`|changer la date|Non|
|`setTime()`|changer l'heure|Non|
|`setTimestamp()`|changer le timestamp|Non|
|`setTimezone()`|changer le fuseau|Non|
|`getTimestamp()`|récupérer le timestamp|Non|
|`getTimezone()`|récupérer le fuseau|Non|
|`getOffset()`|récupérer le décalage|Non|
|`getMicrosecond()`|récupérer les microsecondes|Non|
|`createFromFormat()`|parser une date|—|
|`createFromTimestamp()`|créer depuis timestamp|—|
|`createFromInterface()`|créer depuis une autre date|—|

La documentation officielle précise notamment que `modify()`, `add()`, `sub()` et `setTimezone()` retournent un nouvel objet pour `DateTimeImmutable`. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

## `DateTime`

Les méthodes sont globalement les mêmes :

|Méthode|Rôle|Modifie l'objet ?|
|---|---|---|
|`format()`|formater|Non|
|`modify()`|modifier une date|**Oui**|
|`add()`|ajouter un intervalle|**Oui**|
|`sub()`|soustraire un intervalle|**Oui**|
|`diff()`|calculer une différence|Non|
|`setDate()`|changer la date|**Oui**|
|`setTime()`|changer l'heure|**Oui**|
|`setTimestamp()`|changer le timestamp|**Oui**|
|`setTimezone()`|changer le fuseau|**Oui**|
|`getTimestamp()`|récupérer le timestamp|Non|
|`getTimezone()`|récupérer le fuseau|Non|
|`getOffset()`|récupérer le décalage|Non|
|`createFromFormat()`|parser une date|—|

---

# 34. Bonnes pratiques

## 1. Préférer `DateTimeImmutable`

Pour du nouveau code :

```php
$date = new DateTimeImmutable();
```

est généralement préférable à :

```php
$date = new DateTime();
```

si tu n'as pas spécifiquement besoin d'un objet mutable.

PHP recommande `DateTimeImmutable` lorsque l'on souhaite éviter les modifications accidentelles d'un objet partagé. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP

---

## 2. Toujours penser au fuseau horaire

Pour une application française :

```php
$date = new DateTimeImmutable(
    'now',
    new DateTimeZone('Europe/Paris')
);
```

Ou définir correctement le fuseau par défaut de l'application.

---

## 3. Ne pas mélanger inutilement les chaînes et les objets

À éviter :

```php
$date = '2026-09-30';

$date = date('Y-m-d', strtotime($date));
```

Préférer :

```php
$date = new DateTimeImmutable('2026-09-30');

echo $date->format('Y-m-d');
```

L'objet permet ensuite de continuer à manipuler proprement la date.

---

## 4. Utiliser `createFromFormat()` pour les entrées connues

Si un formulaire fournit :

```text
30/09/2026
```

faire :

```php
$date = DateTimeImmutable::createFromFormat(
    'd/m/Y',
    $input
);
```

plutôt que de supposer que PHP devinera toujours correctement le format.

---

## 5. Ne pas faire confiance aveuglément aux dates utilisateur

Une date provenant d'un formulaire doit être validée.

Exemple :

```php
$date = DateTimeImmutable::createFromFormat(
    '!d/m/Y',
    $input
);

$errors = DateTimeImmutable::getLastErrors();

if (
    $date === false ||
    ($errors !== false && (
        $errors['warning_count'] > 0 ||
        $errors['error_count'] > 0
    ))
) {
    throw new InvalidArgumentException(
        'Date invalide'
    );
}
```

Le `!` dans le format permet notamment de réinitialiser les champs non spécifiés à une valeur de référence plutôt que de les laisser dépendre de l'heure courante.

---

# 35. Résumé

Si tu dois retenir seulement quelques éléments :

## `DateTimeImmutable`

```php
$date = new DateTimeImmutable();
```

Créer une date.

```php
$date->format('d/m/Y');
```

Afficher une date.

```php
$date->modify('+1 day');
```

Créer une nouvelle date à partir de la précédente.

```php
$date->add(new DateInterval('P10D'));
```

Ajouter une durée.

```php
$date->sub(new DateInterval('P10D'));
```

Retirer une durée.

```php
$date->diff($autreDate);
```

Calculer une différence.

```php
$date->setTimezone(
    new DateTimeZone('Europe/Paris')
);
```

Changer de fuseau.

```php
DateTimeImmutable::createFromFormat(
    'd/m/Y',
    '30/09/2026'
);
```

Convertir une chaîne dans un format connu.

---

# `DateTime` vs `DateTimeImmutable`

La phrase à retenir est :

> **`DateTime` modifie l'objet existant ; `DateTimeImmutable` crée un nouvel objet lors des modifications.**

### `DateTime`

```php
$date = new DateTime('2026-09-30');

$date->modify('+1 day');

// $date = 01/10/2026
```

### `DateTimeImmutable`

```php
$date = new DateTimeImmutable('2026-09-30');

$demain = $date->modify('+1 day');

// $date   = 30/09/2026
// $demain = 01/10/2026
```

C'est la différence fondamentale entre les deux classes. P![](https://www.google.com/s2/favicons?domain=https%3A%2F%2Fwww.php.net&sz=128)PHP+1

---

# Fiche mémo

```php
// Maintenant
new DateTimeImmutable();

// Aujourd'hui à minuit
new DateTimeImmutable('today');

// Demain
new DateTimeImmutable('tomorrow');

// Hier
new DateTimeImmutable('yesterday');

// Date précise
new DateTimeImmutable('2026-09-30');

// Date + heure
new DateTimeImmutable('2026-09-30 14:30:00');

// Format français
$date->format('d/m/Y');

// Format SQL
$date->format('Y-m-d H:i:s');

// Année
$date->format('Y');

// Mois
$date->format('m');

// Jour
$date->format('d');

// Heure
$date->format('H:i:s');

// Ajouter
$date->modify('+10 days');

// Soustraire
$date->modify('-2 months');

// Début du mois
$date->modify('first day of this month');

// Fin du mois
$date->modify('last day of this month');

// Prochain lundi
$date->modify('next monday');

// Différence
$date1->diff($date2);

// Timestamp
$date->getTimestamp();

// Fuseau
$date->getTimezone();

// Changer de fuseau
$date->setTimezone(
    new DateTimeZone('Europe/Paris')
);

// Parser
DateTimeImmutable::createFromFormat(
    'd/m/Y',
    '30/09/2026'
);
```