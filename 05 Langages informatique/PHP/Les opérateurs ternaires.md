
### `??` — coalescence nulle

```php
$resultat = $valeur ?? 'valeur par défaut';
```

Signifie :

> Si `$valeur` **existe et n'est pas `null`**, prends `$valeur`, sinon prends `'valeur par défaut'`.

Équivalent :

```php
if (isset($valeur)) {
    $resultat = $valeur;
} else {
    $resultat = 'valeur par défaut';
}
```

Exemples :

```php
$nom = null;
echo $nom ?? 'Inconnu';       // Inconnu

$nom = '';
echo $nom ?? 'Inconnu';       // ''   ← chaîne vide conservée

$nom = 0;
echo $nom ?? 'Inconnu';       // 0    ← 0 conservé
```

**À retenir : `??` teste principalement `null` et l'existence de la variable.**

---

### `?:` — ternaire raccourci

```php
$resultat = $valeur ?: 'valeur par défaut';
```

Signifie :

> Si `$valeur` est considérée comme **vraie**, prends `$valeur`, sinon prends `'valeur par défaut'`.

Équivalent :

```php
$resultat = $valeur ? $valeur : 'valeur par défaut';
```

Exemples :

```php
$nom = null;
echo $nom ?: 'Inconnu';       // Inconnu

$nom = '';
echo $nom ?: 'Inconnu';       // Inconnu

$nombre = 0;
echo $nombre ?: 10;           // 10

$nom = 'Guy';
echo $nom ?: 'Inconnu';       // Guy
```

**À retenir : `?:` teste si la valeur est "truthy" ou "falsy".**

---

### La différence essentielle

|Valeur|`$x ?? 'défaut'`|`$x ?: 'défaut'`|
|---|---|---|
|`null`|défaut|défaut|
|`false`|`false`|défaut|
|`0`|`0`|défaut|
|`''`|`''`|défaut|
|`'Guy'`|`'Guy'`|`'Guy'`|
|`[]`|`[]`|défaut|
|variable inexistante|défaut|⚠️ risque de notice/erreur|

Donc :

**`??` → "est-ce que c'est `null` ?"**

**`?:` → "est-ce que c'est vrai ?"**