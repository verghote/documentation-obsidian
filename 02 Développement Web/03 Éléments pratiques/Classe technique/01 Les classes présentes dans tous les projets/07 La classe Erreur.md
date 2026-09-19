# Documentation de la classe `ClasseTechnique\Erreur`

## Présentation

La classe `Erreur` centralise la gestion des exceptions et des erreurs SQL dans l'application.

Elle assure :

- La **journalisation technique systématique** des erreurs imprévues dans `log/erreur.log`.
    
- L'**absence de journalisation pour les erreurs métier** (`UserException` et contraintes SQL identifiées).
    
- L'**affichage d'un message adapté** à l'utilisateur (message explicite pour une erreur métier, message générique pour une erreur technique).
    
- L'**envoi d'une réponse adaptée au contexte** : JSON (requêtes AJAX / API) ou redirection HTML (`/erreur`).
    

### Principe fondamental

L'application ne gère pas les erreurs script par script. On ne fait pas de `try / catch` systématique pour afficher un message. On laisse remonter les exceptions jusqu'au gestionnaire global.

## Installation et initialisation

### 1. Activer le gestionnaire global

Dans le fichier d'amorçage de l'application (ex. `bootstrap/autoload.php`) :

PHP

```
use ClasseTechnique\Erreur;

Erreur::installerGestionnaire();
```

Cette méthode configure `set_exception_handler()` pour intercepter automatiquement toute exception non attrapée.

### 2. Définir les contraintes SQL spécifiques (Optionnel)

Pour personnaliser les messages d'erreur liés aux contraintes de votre base de données (`UNIQUE`, `CHECK`, clés primaires), chargez votre fichier de configuration :

PHP

```
use ClasseTechnique\Erreur;

$fichierContraintes = RACINE . '/config/contrainte.php';

if (is_file($fichierContraintes)) {
    Erreur::definirLesContraintes(require $fichierContraintes);
}
```

Le fichier `config/contrainte.php` doit retourner un **tableau associatif** :

PHP

```
<?php
// config/contrainte.php

return [
    // 1. Contraintes explicites (UNIQUE, CHECK)
    'uk_projet_nom'    => "Un projet portant ce nom existe déjà.",
    'uk_coureur_email' => "Cette adresse email est déjà utilisée.",
    'ck_projet_dates'  => "La date de fin doit être postérieure à la date de début.",

    // 2. Clés primaires spécifiques par table (Format : primary@nom_table)
    'primary@coureur'   => "Ce coureur est déjà inscrit.",
    'primary@categorie' => "Ce code catégorie existe déjà.",

    // 3. Clé primaire par défaut (fallback si aucune table/contexte n'est trouvé)
    'primary'           => "Cet identifiant existe déjà."
];
```

## Gestion des exceptions dans le code

### Erreurs métier (`UserException`)

Pour signaler une erreur d'un contrôle applicatif (saisie incorrecte, règle de gestion non respectée), le développeur lève une `UserException`.

PHP

```
if ($ancienMotDePasse !== $motDePasseBD) {
    throw new UserException("Le mot de passe actuel est incorrect.");
}
```

**Comportement :**

- Le message réel est transmis à l'utilisateur (`"Le mot de passe actuel est incorrect."`).
    
- **Aucun journal technique n'est écrit** (il s'agit d'une utilisation normale du système).
    
- Code HTTP retourné : celui défini dans la `UserException` (par défaut `400`).
    

### Erreurs techniques (`Throwable`, `Exception`)

Si une exception standard est levée (ou une erreur système PHP) :

PHP

```
throw new Exception("Impossible de contacter le serveur distant.");
```

**Comportement :**

- L'erreur est **journalisée** dans `log/erreur.log` avec son type, son message, son code, le fichier et la ligne.
    
- L'utilisateur reçoit le message générique de sécurité :
    
    _« Une erreur technique est survenue, veuillez réessayer ultérieurement. »_
    
- Code HTTP retourné : `500`.
    

## Gestion automatique des erreurs SQL (`PDOException`)

Lorsqu'une `PDOException` remonte, la classe `Erreur` l'analyse selon une cascade de priorités :

```
1. Exception SQLState 45000 (SIGNAL SQL personnalisé)
   ↓
2. Contrainte applicative explicite dans config/contrainte.php
   ↓
3. Doublon sur Clé Primaire / Code 1062 (recherche primary@table, primary@contexte ou primary)
   ↓
4. Contrainte CHECK générique (codes MySQL 3819, MariaDB 4025)
   ↓
5. Code SQL MySQL/MariaDB connu (1062 générique, 1048, 1451, etc.)
   ↓
6. Erreur SQL inconnue (considérée comme erreur technique)
```

### Table des codes SQL natifs pris en charge

Pour les erreurs courantes sans contrainte nommée, la classe fournit des messages par défaut :

|**Code SQL**|**Cause**|**Message restitué à l'utilisateur**|
|---|---|---|
|**1062**|Doublon (clé unique)|_Enregistrement déjà existant._|
|**1048**|Champ `NOT NULL` manquant|_Une information obligatoire est manquante._|
|**1406**|Donnée trop longue|_Une information est trop longue._|
|**1366**|Format de donnée invalide|_Format de donnée invalide._|
|**1452**|Clé étrangère introuvable|_Donnée invalide (référence inexistante)._|
|**1451**|Suppression bloquée (FK)|_Suppression impossible : donnée utilisée._|
|**3819 / 4025**|Contrainte `CHECK` violée|_Une valeur saisie ne respecte pas une règle de validation._|

### Gestion dynamique des erreurs de clé primaire (`PRIMARY`)

Lors d'une violation de clé primaire (code SQL `1062`), la classe tente de déterminer automatiquement la table concernée pour renvoyer un message contextuel :

1. **Via l'index SQL** : extrait le nom de la table si le message MySQL contient une clé qualifiée (`db.table.PRIMARY`) et recherche `primary@nom_table` dans `config/contrainte.php`.
    
2. **Via l'URL** : analyse l'URI de la requête (ex. `/coureur/maj/ajax/ajouter.php` de laquelle elle déduit `primary@coureur`).
    
3. **Fallback** : utilise la clé `'primary'` définie dans `config/contrainte.php` ou le message générique _"Enregistrement déjà existant."_.
    

> **Remarque sur la journalisation :**
> 
> Les cas 1 à 5 sont considérés comme des erreurs de saisie ou de logique métier attendues : **ils ne génèrent pas de journal d'erreur**. Seul le cas 6 (erreur SQL inconnue ou panne de SGBD) déclenche une journalisation technique et affiche le message générique système.

## Restitution des réponses (HTML vs JSON)

La classe détermine automatiquement le format de réponse grâce à la méthode interne `getTypeReponse()` :

- **Format JSON** : si l'en-tête `X-Requested-With: XMLHttpRequest` est présent (requête AJAX) ou si l'en-tête `Accept` contient `application/json`.
    
    - Nettoie les tampons de sortie (`ob_end_clean()`).
        
    - Déclenche `ReponseJson::envoyerErreur($message, $codeHttp)`.
        
- **Format HTML** : navigation classique.
    
    - Nettoie les tampons de sortie.
        
    - Ouvre/reprend la session.
        
    - Stocke le message dans `$_SESSION['erreur']`.
        
    - Redirige vers `/erreur`.
        

## Méthodes utilitaires

### Libellé d'erreur HTTP (`getErreurHttp`)

Permet de récupérer la traduction texte d'un code de statut HTTP pour l'affichage de la page `/erreur`.

PHP

```
echo Erreur::getErreurHttp(404); // Affiche "Page non trouvée"
echo Erreur::getErreurHttp(500); // Affiche "Erreur interne du serveur"
```

## Résumé des bonnes pratiques

|**Action**|**Bonne pratique**|**À éviter**|
|---|---|---|
|**Erreur de saisie / métier**|`throw new UserException("Message explicite");`|`throw new Exception("Message métier");` _(masquerait le message)_|
|**Erreur technique imprévue**|Laisser l'exception remonter ou `throw new Exception(...)`|Afficher directement `echo $e->getMessage();`|
|**Gestion des Blocs**|Laisser remonter l'exception jusqu'au gestionnaire global|Faire des `try/catch` vides ou ré-afficher du HTML manuellement|
|**Contraintes de BDD**|Déclarer les noms de contraintes et les clés `primary@table` dans `config/contrainte.php`|Modifier le code source de la classe `Erreur`|