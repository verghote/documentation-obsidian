## Présentation

La classe `Std` est une classe utilitaire statique regroupant un ensemble de fonctions génériques de validation, de nettoyage, de conversion et de manipulation de données.

Elle agit comme une boîte à outils centrale réutilisable dans toute l’application. Aucune instance n’est créée : toutes les méthodes sont statiques.

## Rôle dans l’architecture

La classe `Std` intervient en amont des traitements métier. Elle permet de garantir :

- La cohérence des données entrantes ;
- La validation des formats utilisateur ;
- La normalisation des chaînes de caractères et des titres ;
- La purification du contenu HTML contre les failles XSS ;
- La conversion des formats de données ;
- L’accès simplifié à certaines opérations système.
    

```
Entrée utilisateur (Requete / Controller)
      │
      ▼
Std (validation / nettoyage / purification)
      │
      ▼
Classes métier (Annonce, Table, etc.)
      │
      ▼
Base de données / Logique applicative
```

## Philosophie générale

La classe repose sur un principe simple : **Toute donnée extérieure doit être validée ou normalisée avant utilisation.**

Elle réduit donc :

- La duplication de code ;
- Les erreurs de validation ;
- Les incohérences de format ;
- Les risques de sécurité (injections XSS, en-têtes corrompus).
    
## Sécurité et purification HTML

### Purification de contenu HTML

#### `Std::nettoyerHtml(?string $valeur): string`

- **Rôle :** Nettoie et sécurise du code HTML fourni par l'utilisateur (ex. descriptions, articles) pour prévenir les injections XSS tout en préservant la mise en forme légitime.
    
- **Fonctionnement :** Repose sur l'outil `HTMLPurifier`.
    
    - Instancie de manière unique (Singleton) un objet `HTMLPurifier` configuré.
    - Supprime les balises et attributs dangereux (`<script>`, `<iframe>`, `onclick`, `javascript:`).
    - Convertit le HTML sous le Doctype `XHTML 1.0 Transitional`.
    - Sécurise les liens externes (injection automatique de `rel="nofollow noreferrer noopener"` via la directive `HTML.TargetBlank`).
    - Gère le stockage en cache de la configuration (`/cache/htmlpurifier` ou dossier temporaire système).
        
- **Exemple :**
    ```php
    $htmlInfecte = "Tentative <script>alert('xss')</script><b>Texte</b>";
    $htmlPropre = Std::nettoyerHtml($htmlInfecte);
    // Résultat : "Tentative <b>Texte</b>"
    ```

## Nettoyage et normalisation des chaînes

### Normalisation de titre

#### `Std::nettoyerTitre(string $titre): string`

- **Rôle :** Nettoie et normalise le titre ou la dénomination d'un objet métier (ex. nom d'une annonce).
    
- **Fonctionnement :**
    
    1. Utilise l'extension `Normalizer` (formule `FORM_C`) pour unifier la représentation Unicode des caractères accentués (forme pré-composée NFC).
    2. Passe la chaîne dans `Std::supprimerEspace()` pour éliminer tout superflu.

+ **La normalisation Unicode** (`Normalizer::normalize`)

		En UTF-8, certains caractères accentués peuvent être codés de deux manières différentes dans la mémoire :

			- Forme décomposée (NFD) : La lettre `e` + un caractère d'accent aigu séparé (`e` + `´`).
			    
			- Forme composée (NFC) : Un seul caractère pré-composé (`é`).
    
			La fonction `Normalizer::normalize(..., \Normalizer::FORM_C)` **uniformise tous les caractères accentués vers leur forme composée (NFC)**.

	**Pourquoi c'est important ?**
	Sans cela, deux titres visuellement identiques (`Écran` et `Écran`) pourraient être considérés comme différents par MySQL ou PHP, ce qui poserait des problèmes lors d'une recherche, d'un tri ou d'un test d'unicité.

- **Exemple :**
    ```php
    $titre = "   Vente   d'un  écran   4K   ";
    $titrePropre = Std::nettoyerTitre($titre);
    // Résultat : "Vente d'un écran 4K"
    ```
    
### Suppression des espaces

#### `Std::supprimerEspace(string $valeur): string`

- **Rôle :** Normalise les espaces dans une chaîne.
    
- **Fonctionnement :**
    - Remplace les suites d'espaces multiples (y compris tabulations, retours à la ligne et espaces Unicode/insécables) par un unique espace standard (`/[\s\p{Z}]+/u`).
    - Supprime les espaces au début et à la fin de la chaîne via `trim()`.
- **Exemple :**
    ```php
    Std::supprimerEspace("  Bonjour    le   monde  ");
    // Résultat : "Bonjour le monde"
    ```
### Suppression des accents

#### `Std::supprimerAccent(string $valeur): string`

- **Rôle :** Convertit une chaîne accentuée en version ASCII.
    
- **Fonctionnement :**
    1. Utilise le `Transliterator` ICU (`Any-Latin; Latin-ASCII`) si l'extension `intl` est disponible.
    2. Applique un tableau de correspondance manuel en fallback.
- **Exemple :**
    ```php
    Std::supprimerAccent("Élévation");
    // Résultat : "Elevation"
    ```

## Verification de présence de paramètres

### Méthode `existe()`

#### `Std::existe(string ...$champs): bool`

- **Rôle :** Vérifie que plusieurs clés existent dans la superglobale `$_REQUEST`.
    
- **Fonctionnement :** Parcourt tous les champs demandés et retourne `false` dès qu’un champ est absent.
    
- **Exemple :**
    ```php
    if (Std::existe('primaryKey', 'columns')) {
        // Traitement du contrôleur
    }
    ```

## Validation des titres et noms

### Validation de titre

#### `Std::titreValide(string $titre, int $longueurMax = 255): bool`

- **Rôle :** Contrôle la validité d'un titre nettoyé.
    
- **Critères :**
    - Longueur non vide et inférieure ou égale à `$longueurMax` caractères (UTF-8).
    - Doit contenir au moins un caractère alphanumérique (`[\p{L}\p{N}]`).
    - Ne doit contenir aucun caractère de contrôle invisible (`[\p{Cc}\p{Cf}]`).

### Validation des noms

#### `Std::nomSansAccentValide(string $valeur): bool`

- **Rôle :** Vérifie un nom composé uniquement de caractères ASCII de base.
    
- **Critères :** Lettres de `a` à `z` (insensible à la casse), tirets, espaces et apostrophes.
    

#### `Std::nomAvecAccentValide(string $valeur): bool`

- **Rôle :** Vérifie un nom pouvant contenir des caractères accentués Unicode.
    
- **Critères :** Regex PCRE `/^[\p{L}\p{M}]+(?:[ '\x{2019}-][\p{L}\p{M}]+)*$/u`.
    
## Gestion des dates

### Encodage date française → MySQL

#### `Std::encoderDate(string $date): string`

- **Rôle :** Convertit une date au format `jj/mm/aaaa` vers `aaaa-mm-jj`.
    
- **Sécurité :** Vérifie au préalable la validité de la date via `dateFrValide()`. Retourne une chaîne vide si invalide.

### Décodage MySQL → français

#### `Std::decoderDate(string $date): string`

- **Rôle :** Convertit une date au format `aaaa-mm-jj` vers `jj/mm/aaaa`.
    
- **Sécurité :** Vérifie la validité au préalable via `dateMysqlValide()`.
### Validation de date française

#### `Std::dateFrValide(string $valeur): bool`

- **Critères :** Format strict `jj/mm/aaaa`, existence réelle de la date (`checkdate`) et année strictement supérieure à `1900`.

### Validation de date MySQL

#### `Std::dateMysqlValide(string $valeur): bool`

- **Critères :** Format strict `aaaa-mm-jj`, existence réelle de la date (`checkdate`) et année strictement supérieure à `1900`.
    
## Validation Web (URL & Email)

### Validation et accessibilité d'URL

#### `Std::urlValide(string $valeur): bool`

- **Rôle :** Vérifie la syntaxe d'une URL via `FILTER_VALIDATE_URL`.

#### `Std::urlAccessible(string $valeur): bool`

- **Rôle :** Vérifie la syntaxe ET s'assure que le serveur distant répond.
    
- **Fonctionnement :** Exécute une requête HTTP HEAD via cURL avec un timeout de 3 secondes et vérifie que le code de réponse HTTP est compris entre 200 et 399.

### Validation d'email et de domaine

#### `Std::emailValide(string $valeur): bool`

- **Rôle :** Vérifie la syntaxe de l'adresse email via `FILTER_VALIDATE_EMAIL`.
    
#### `Std::domaineEmailExiste(string $valeur): bool`

- **Rôle :** Vérifie la syntaxe de l'email ET l'existence d'enregistrements DNS (`MX` ou `A`) pour le domaine associé via `checkdnsrr`.
    
## Validation de sécurité et formats français

### Mot de passe

#### `Std::passwordValide(string $valeur, int $longueur = 8): bool`

- **Critères :**
    - Longueur minimale (par défaut 8 caractères UTF-8).
    - Au moins une lettre minuscule.
    - Au moins une lettre majuscule.
    - Au moins un chiffre.
    - Au moins un caractère spécial (`[\W_]`).
### Formats français

- **Code postal :** `Std::codePostalValide(string $valeur)` — Format à 5 chiffres (ex. `80000`, gestion DOM-TOM `97x`/`98x`).
- **Téléphone mobile :** `Std::mobileValide(string $valeur)` — Format à 10 chiffres débutant par `06` ou `07`.
- **Téléphone fixe :** `Std::fixeValide(string $valeur)` — Format à 10 chiffres débutant par `01` à `05` ou `09`.
    
### Format de temps

#### `Std::tempsValide(string $valeur): bool`

- **Critères :** Format strict `hh:mm:ss` (ex. `14:30:00`).
    
### Validation numérique

- **Entier :** `Std::nombreEntierValide($valeur)` — Accepte le type `int` ou une chaîne numérique entière (`^-?\d+$`).
- **Nombre réel :** `Std::nombreReelValide($valeur)` — Utilise `is_numeric()`.

## Gestion des fichiers

### Méthode `getLesFichiers()`

#### `Std::getLesFichiers(string $rep, array $extensions = [], string $order = "a"): array`

- **Rôle :** Retourne la liste des fichiers d'un dossier avec filtrage optionnel par extension.
    
- **Fonctionnement :**
    - Ignore les éléments `.` et `..` ainsi que les sous-dossiers.
    - Filtre par extensions (insensible à la casse).
    - Applique un tri naturel (`natcasesort`).
    - Permet un tri croissant (`"a"`) ou décroissant (`"d"`).
## Exemple d'utilisation globale dans un contrôleur AJAX

```php
<?php
declare(strict_types=1);

use ClasseMetier\Annonce;
use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;
use ClasseTechnique\Std;

require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/bootstrap.php';

$id = Requete::postString('primaryKey');
$columns = Requete::postArray('columns');

// 1. Normalisation et nettoyage des données entrantes
if (isset($columns['nom'])) {
    $columns['nom'] = Std::nettoyerTitre($columns['nom']);
}

if (isset($columns['description'])) {
    $columns['description'] = Std::nettoyerHtml($columns['description']);
}

// 2. Traitement métier
$annonce = new Annonce();
$resultat = $annonce->modify($id, $columns);

// 3. Envoi de la réponse
if ($resultat === true) {
    ReponseJson::envoyerMessage("Annonce modifiée avec succès");
}

ReponseJson::envoyerLesErreurs($annonce->getErrors());
```
## Position dans l’architecture globale

Plaintext

```
  Vue / Client (AJAX, Formulaire)
                 │
                 ▼
     Contrôleur (Requete / AJAX)
                 │
                 ▼
  Std (Validation, Nettoyage HTML, Titre)
                 │
                 ▼
    Classe Métier (Annonce, Table)
                 │
                 ▼
          Base de données
```

## Avantages de la classe

1. **Centralisation des validations :** Un point unique de contrôle pour toute l'application.
2. **Sécurité renforcée :** Protection native contre les failles XSS (via `HTMLPurifier`) et les corruptions de données.
3. **Pérennité des données :** Normalisation Unicode et gestion propre des espaces/accents pour des recherches et tris fiables en base de données.
4. **Réduction de la redondance :** Évite de réécrire les regex de validation dans chaque contrôleur.
5. **Robustesse et tolérance :** Prise en compte des modules PHP disponibles (ex. ICU `Transliterator`, `Normalizer`) avec solutions de repli.