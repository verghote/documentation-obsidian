# Présentation

La classe `Std` est une classe utilitaire statique regroupant un ensemble de fonctions génériques de **validation**, de **nettoyage**, de **conversion** et de **manipulation de données**.

Elle agit comme une boîte à outils centrale réutilisable dans toute l’application.

Aucune instance n’est créée : toutes les méthodes sont statiques.

---

# Rôle dans l’architecture

La classe `Std` intervient en amont des traitements métier.

Elle permet de garantir :

- la cohérence des données entrantes ;
    
- la validation des formats utilisateur ;
    
- la normalisation des chaînes ;
    
- la conversion des formats de données ;
    
- l’accès simplifié à certaines opérations système.
    

```text
Entrée utilisateur
      │
      ▼
Std (validation / nettoyage)
      │
      ▼
Classes métier (Table, etc.)
      │
      ▼
Base de données / logique applicative
```

---

# Philosophie générale

La classe repose sur un principe simple :

> Toute donnée extérieure doit être validée ou normalisée avant utilisation.

Elle réduit donc :

- la duplication de code ;
    
- les erreurs de validation ;
    
- les incohérences de format.
    

---

# Vérification de présence de paramètres

## Méthode `existe()`

```php
Std::existe('id', 'nom');
```

### Rôle

Vérifie que plusieurs clés existent dans `$_REQUEST`.

### Fonctionnement

- Parcourt tous les champs demandés
    
- Retourne `false` dès qu’un champ est absent
    
- Retourne `true` si tous sont présents
    

### Utilisation

Principalement utilisée pour sécuriser les contrôleurs.

---

# Nettoyage des chaînes

## Suppression des espaces

```php
Std::supprimerEspace(string $valeur): string
```

### Rôle

Normalise les espaces dans une chaîne :

- supprime les espaces en début et fin
    
- remplace les espaces multiples par un seul espace
    

### Exemple

```text
"  Bonjour    le   monde  "
→ "Bonjour le monde"
```

---

## Suppression des accents

```php
Std::supprimerAccent(string $valeur): string
```

### Rôle

Convertit une chaîne accentuée en version ASCII.

### Fonctionnement

Deux modes :

1. **Transliterator ICU** (si disponible)
    
2. **Fallback manuel** via tableau de correspondance
    

### Exemple

```text
"Élévation"
→ "Elevation"
```

---

# Gestion des dates

## Encodage date française → MySQL

```php
Std::encoderDate('31/12/2026')
→ '2026-12-31'
```

### Sécurité

La méthode vérifie d’abord la validité de la date française.

---

## Décodage MySQL → français

```php
Std::decoderDate('2026-12-31')
→ '31/12/2026'
```

---

## Validation date française

```php
Std::dateFrValide('31/12/2026')
```

Vérifie :

- format `jj/mm/aaaa`
    
- cohérence réelle de la date
    
- année > 1900
    

---

## Validation date MySQL

```php
Std::dateMysqlValide('2026-12-31')
```

Vérifie :

- format `aaaa-mm-jj`
    
- cohérence réelle
    
- année > 1900
    

---

# Validation d’URL

## Méthode `urlValide()`

```php
Std::urlValide(string $valeur)
```

### Étapes

1. Vérification syntaxique (`FILTER_VALIDATE_URL`)
    
2. Requête HTTP HEAD via cURL
    
3. Vérification du code HTTP
    

### Résultat

- `true` → URL accessible
    
- `false` → URL invalide ou inaccessible
    

---

# Validation email

```php
Std::emailValide(string $valeur)
```

### Contrôles

- format email valide
    
- existence du domaine (MX ou A record DNS)
    

---

# Validation mot de passe

```php
Std::passwordValide(string $valeur, int $longueur = 8)
```

### Critères

Un mot de passe valide doit contenir :

- une minuscule
    
- une majuscule
    
- un chiffre
    
- un caractère spécial
    
- une longueur minimale
    

---

# Validation des formats français

## Code postal

```php
Std::codePostalValide(string $valeur)
```

---

## Téléphone mobile

```php
Std::mobileValide(string $valeur)
```

Formats acceptés :

- 06xxxxxxxx
    
- 07xxxxxxxx
    

---

## Téléphone fixe

```php
Std::fixeValide(string $valeur)
```

Formats commençant par :

- 01 à 05
    
- 09
    

---

# Validation du temps

```php
Std::tempsValide('14:30:59')
```

Format strict :

```text
hh:mm:ss
```

---

# Validation des noms

## Nom simple

```php
Std::nomValide(string $valeur)
```

Accepte uniquement :

- lettres ASCII
    
- espaces simples
    
- tirets
    
- apostrophes
    

---

## Nom avec accents

```php
Std::nomAvecAccentValide(string $valeur)
```

Identique mais avec support des caractères accentués.

---

# Validation numérique

## Entier

```php
Std::nombreEntierValide($valeur)
```

Accepte :

- int
    
- chaîne numérique entière
    

---

## Nombre réel

```php
Std::nombreReelValide($valeur)
```

Utilise `is_numeric()`.

---

# Gestion des fichiers

## Méthode `getLesFichiers()`

```php
Std::getLesFichiers(string $rep, array $extensions = [], string $order = "a")
```

### Rôle

Retourne la liste des fichiers d’un dossier avec filtrage optionnel.

---

## Fonctionnement

- ignore `.` et `..`
    
- ignore les dossiers
    
- filtre par extensions si demandé
    
- trie naturellement (`natcasesort`)
    
- permet tri croissant ou décroissant
    

---

## Exemple

```php
Std::getLesFichiers('/images', ['jpg', 'png']);
```

---

# Exemple d’utilisation globale

```php
if (
    Std::existe('email') &&
    Std::emailValide($_POST['email'])
) {
    $email = Std::supprimerEspace($_POST['email']);
}
```

---

# Position dans l’architecture

```text
Vue / Controller
      │
      ▼
Std (validation / normalisation)
      │
      ▼
Classe Table / Métier
      │
      ▼
Base de données
```

---

# Avantages de la classe

- Centralisation des validations
    
- Réduction du code redondant
    
- Cohérence des formats
    
- Sécurisation des entrées utilisateur
    
- Réutilisation dans toute l’application
    
- Tolérance aux environnements (ICU / fallback)
    
- Facilité de maintenance
    

---

# Conclusion

La classe `Std` constitue une **bibliothèque utilitaire essentielle** du système.

Elle garantit que toutes les données manipulées par l’application sont :

- propres
    
- cohérentes
    
- validées
    
- exploitables sans risque