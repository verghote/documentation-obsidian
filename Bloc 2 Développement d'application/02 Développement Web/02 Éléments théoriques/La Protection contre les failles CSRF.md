Les attaques de type **CSRF** (_Cross-Site Request Forgery_ ou « Falsification de requête intersites ») consistent à piéger un utilisateur authentifié pour lui faire exécuter, à son insu, des actions critiques sur une application (création, modification, suppression de données).

Pour se prémunir de cette faille, la solution standard consiste à utiliser un **jeton CSRF (CSRF Token)** : un identifiant unique, imprévisible et à durée de vie limitée, généré par le serveur et associé à la session de l'utilisateur.

## 1. Principe général de fonctionnement

Toutes les pages n'ont pas besoin d'un jeton CSRF. 
Une page contenant uniquement de la consultation (lecture seule) n'en nécessite généralement pas. 
En revanche, le jeton devient obligatoire dès qu'une action modifie l'état de l'application :

- Une création ;
- Une modification ;
- Une suppression ;
- Un appel AJAX protégé.

### Le cycle de vie du jeton

```
┌─────────────────────────────────────────────────────────────────────────┐
│ 1. GÉNÉRATION ET INJECTION (Serveur PHP -> Page HTML)                   │
└─────────────────────────────────────────────────────────────────────────┘
   [ Contrôleur ]  ──( $page->avecJeton() )──>  [ Vue PHP / Template ]
                                                       │
                                            ( Jeton::creer() )
                                                       │
                                                       ▼
   Génération du HTML envoyé au navigateur :
   <meta name="csrf-token" content="abc123xyz...">

───────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────┐
│ 2. LECTURE ET TRANSMISSION (Client JS -> Requête AJAX)                  │
└─────────────────────────────────────────────────────────────────────────┘
   [ Document HTML ]
          │
  ( document.querySelector )
          │
          ▼
   [ fonction.ajax.js ] ──( Ajout de l'en-tête )──> En-têtes HTTP :
                                                    Accept: application/json
                                                    X-CSRF-Token: abc123xyz...

───────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────┐
│ 3. VÉRIFICATION DE SÉCURITÉ (Serveur PHP / API)                         │
└─────────────────────────────────────────────────────────────────────────┘
   [ Script PHP / Action ] 
          │
          ├─► 1. Lit l'entête : $_SERVER['HTTP_X_CSRF_TOKEN']
          ├─► 2. Compare avec le jeton stocké en SESSION
          │
          ├── [ Valide ]   ──► Exécute l'action (Création/Modif/Suppr)
          └── [ Invalide ] ──► Rejette la requête (Erreur 403 Forbidden)
```

## 2. Rôle de chaque composant architecturel

Pour mettre en œuvre cette protection de manière propre et maintenable, la responsabilité est découpée entre quatre composants clés :

### A. Le Contrôleur PHP (`$page->avecJeton()`)

Le contrôleur est le chef d'orchestre. C'est lui qui décide si la page consultée nécessite une protection. Si la page comporte un formulaire ou des interactions AJAX modifiant des données, il informe l'objet `$page` qu'un jeton devra être généré.

### B. Le Template / Vue (`<meta name="csrf-token">`)

Le fichier de vue (`view/interface.php`) vérifie si le contrôleur a demandé un jeton via `$page->necessiteUnJeton()`. Si oui, il fait appel à la classe utilitaire `Jeton::creer()`. Le jeton ainsi généré est injecté directement dans l'en-tête du document HTML sous la forme d'une balise `<meta>` :

HTML

```
<meta name="csrf-token" content="d9a8f7e6c5b4a3...">
```

> **Pourquoi une balise `<meta>` ?**
> 
> Placer le jeton dans le `<head>` du document le rend immédiatement accessible à l'ensemble de vos scripts JavaScript, sans polluer le corps (`<body>`) de la page ni multiplier les champs cachés dans chaque formulaire.

### C. Le script client JavaScript (`fonction.ajax.js`)

Lorsqu'un script navigateur exécute un appel AJAX, la fonction générique `appelAjax()` va lire la balise `<meta>` au chargement :

JavaScript

```
const _csrfToken = document.querySelector('meta[name="csrf-token"]')?.content ?? null;
```

Si ce jeton existe, il est automatiquement greffé à l'en-tête HTTP de la requête sortante sous le nom `X-CSRF-Token`. Grâce à cette automatisation, le développeur frontal n'a pas à se soucier de réinjecter manuellement le jeton à chaque appel API.

### D. Le script cible / Action PHP (Vérification)

À la réception d'une requête HTTP modifiante, le script PHP de destination (ou un middleware de sécurité) extrait l'en-tête `X-CSRF-Token` transmis et le compare avec la valeur conservée en session serveur (`$_SESSION['csrf_token']`).

- **Si les jetons correspondent :** La requête est légitime, le traitement continue.
    
- **Si le jeton est absent ou incorrect :** L'action est immédiatement bloquée et le serveur retourne un code d'erreur HTTP **`403 Forbidden`**. Un site tiers distant ne pouvant pas lire la balise `<meta>` du domaine protégé, il lui est impossible de contrefaire cette en-tête HTTP.