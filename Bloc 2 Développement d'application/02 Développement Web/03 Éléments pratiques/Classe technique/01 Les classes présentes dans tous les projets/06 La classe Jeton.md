# Présentation

La classe `Jeton` assure la protection de l'application contre les attaques **CSRF** (_Cross-Site Request Forgery_).

Elle met en œuvre le mécanisme du **Synchronizer Token Pattern**, utilisé pour sécuriser les requêtes HTTP modifiant des données :

- POST ;
- PUT ;
- PATCH ;
- DELETE.

La classe est entièrement statique : aucun objet n'est créé.

---

# Rôle dans l'architecture

Les opérations sensibles doivent être protégées contre les requêtes provenant d'un site tiers.

La classe `Jeton` intervient entre le navigateur et les scripts de traitement.

```text
Navigateur
     │
     │ POST AJAX
     ▼
 Jeton::verifier()
     │
     ├── Jeton valide
     │        │
     │        ▼
     │  Traitement PHP
     │
     └── Jeton invalide
              │
              ▼
      HTTP 403 + UserException
```

Elle garantit qu'une requête provient bien d'une page générée par l'application.

---

# Le principe du CSRF

Une attaque CSRF consiste à exploiter une session déjà ouverte.

Exemple :

1. l'utilisateur est connecté à l'application ;
2. il visite ensuite un site malveillant ;
3. ce site tente d'envoyer une requête vers l'application ;
4. le navigateur transmet automatiquement les cookies de session ;
5. sans protection, l'application pourrait exécuter l'action.

Le jeton CSRF empêche ce scénario en ajoutant une preuve supplémentaire que seule l'application peut générer.

# Principe de fonctionnement

Le mécanisme repose sur un secret partagé entre le serveur et le navigateur.

Le déroulement est le suivant :

```text
1. Création du jeton
        ▼
Stockage en session PHP
        ▼
Injection dans la page HTML
        ▼
Lecture par JavaScript
        ▼
Envoi dans le header HTTP
        ▼
Jeton::verifier()
        ▼
Comparaison avec la session
```

La requête est acceptée uniquement si les deux valeurs correspondent.

# Pourquoi utiliser un header HTTP ?

Le jeton est transmis dans un header personnalisé :

```text
X-CSRF-Token
```

Exemple :

```http
POST /ajax/suppression.php

X-CSRF-Token: 8a53...
```

Un site tiers peut créer un formulaire HTML vers votre application, mais il ne peut pas ajouter librement ce type de header personnalisé.

Cette restriction des navigateurs constitue une protection efficace contre les attaques CSRF classiques.

# Pourquoi ne pas utiliser un cookie ?

Le jeton n'est volontairement pas stocké dans un cookie.

Deux possibilités existent :

- un cookie `HttpOnly` n'est pas accessible par JavaScript ;
- un cookie accessible par JavaScript peut être récupéré par un script malveillant en cas de faille XSS.

Le stockage en session associé à un header HTTP permet de conserver un mécanisme simple et robuste.

---

# Création d'un jeton

La méthode :

```php
Jeton::creer()
```

retourne un jeton CSRF valide.

Prototype :

```php
public static function creer(): string
```

Le jeton est généré grâce à :

```php
random_bytes(32)
```

puis converti en représentation hexadécimale.

Le résultat obtenu possède 64 caractères.

Exemple :

```text
5fbe908d6f74a8c3...
```

---

# Réutilisation du jeton

La classe ne génère pas un nouveau jeton à chaque affichage de page.

Si un jeton existe déjà dans la session, il est réutilisé.

Cette stratégie évite les problèmes liés aux onglets multiples.

Exemple :

```text
Onglet A
    │
    │ reçoit jeton X
    │
Onglet B
    │
    │ utilise jeton X
    │
Les deux onglets restent valides
```

La réutilisation garantit une meilleure expérience utilisateur.

---

# Durée de vie du jeton

Le jeton n'a pas de durée propre.

Il reste valide :

- pendant toute la durée de la session PHP ;
- jusqu'à la fermeture de session ;
- ou jusqu'à sa suppression volontaire.

La gestion de l'expiration est donc entièrement confiée au mécanisme de session.

Pour supprimer volontairement un jeton :

```php
Jeton::supprimer();
```

Cette méthode est notamment utilisée lors d'une déconnexion.

---

# Création conditionnelle du jeton

Un jeton n'est pas créé systématiquement.

Seules les pages contenant des actions nécessitant une protection CSRF le demandent.

Exemple :

```php
$page
    ->setAvecToken()
    ->afficher();
```

Une page uniquement destinée à l'affichage :

```php
$page
    ->setDonnee('annonces', $annonces)
    ->afficher();
```

ne génère aucun jeton.

Cette approche évite les traitements inutiles.

---

# Vérification d'un jeton

La méthode :

```php
Jeton::verifier()
```

contrôle qu'une requête est authentique.

Trois vérifications sont réalisées.

---

# Étape 1 : présence du jeton en session

La classe vérifie qu'un jeton existe toujours côté serveur.

Les causes possibles d'absence :

- session expirée ;
- déconnexion ;
- destruction de session ;
- ancienne page conservée trop longtemps.

Dans ce cas :

```text
HTTP 403 Forbidden
```

est retourné.

---

# Étape 2 : présence du header HTTP

Le navigateur doit transmettre :

```text
X-CSRF-Token
```

L'absence de ce header indique généralement :

- un problème JavaScript ;
- une requête construite manuellement ;
- une tentative de contournement.

La requête est refusée.

---

# Étape 3 : comparaison des valeurs

Le jeton reçu est comparé au jeton stocké en session.

La comparaison utilise :

```php
hash_equals()
```

Cette fonction protège contre les attaques par analyse temporelle (_Timing Attacks_).

---

# Refus d'une requête

Toutes les erreurs passent par la méthode privée :

```php
refuser()
```

Cette méthode :

- positionne le code HTTP `403 Forbidden` ;
- lève une `UserException`.

La gestion globale des erreurs prend ensuite le relais.

---

# Intégration dans une page

Dans le template `interface.php` :

```php
<?php if ($page->avecToken()) : ?>

<meta name="csrf-token" content="<?= Jeton::creer() ?>">

<?php endif; ?>
```

Le jeton est injecté uniquement si la page l'a demandé.

---

# Utilisation côté JavaScript

Le JavaScript récupère le jeton :

```javascript
const token = document
    .querySelector('meta[name="csrf-token"]')
    .content;
```

Puis l'ajoute aux requêtes AJAX :

```javascript
fetch(url, {
    method: "POST",
    headers: {
        "X-CSRF-Token": token
    }
});
```

---

# Utilisation dans un endpoint

Avant tout traitement modifiant :

```php
Controle::verifier('post', 'jeton');
```

La vérification intervient avant toute modification de données.

Exemple :

```text
Contrôle méthode HTTP
        │
        ▼
Contrôle CSRF
        │
        ▼
Validation des données
        │
        ▼
Modification base de données
```

---

# Cycle complet

```text
Utilisateur
      │
      ▼
Affichage page protégée
      │
      ▼
Page::setAvecToken()
      │
      ▼
interface.php
      │
      ▼
Jeton::creer()
      │
      ▼
Stockage en session
      │
      ▼
<meta csrf-token>
      │
      ▼
JavaScript
      │
      ▼
Requête AJAX
      │
      ▼
Header X-CSRF-Token
      │
      ▼
Jeton::verifier()
      │
      ├── valide
      │      │
      │      ▼
      │ Traitement PHP
      │
      └── invalide
             │
             ▼
       HTTP 403 Forbidden
```

---

# Avantages de cette architecture

- Conforme au **Synchronizer Token Pattern**.
- Protection efficace contre les attaques CSRF.
- Génération cryptographiquement sûre avec `random_bytes()`.
- Comparaison sécurisée avec `hash_equals()`.
- Compatible avec plusieurs onglets ouverts.
- Création du jeton uniquement lorsque nécessaire.
- Séparation claire des responsabilités :
  - `Page` indique le besoin ;
  - `Jeton` gère la sécurité ;
  - `interface.php` génère le HTML ;
  - `Controle` applique les contrôles.
- Implémentation simple, statique et réutilisable dans toute l'application.