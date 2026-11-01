## 1. Introduction

Dans les systèmes d'information modernes, la gestion de l'authentification, des sessions et des autorisations constitue un enjeu majeur pour garantir la sécurité des utilisateurs et des données.

Les jetons (_tokens_) sont des valeurs utilisées par les applications pour identifier une session, transporter des informations d'authentification ou vérifier qu'une requête respecte certaines conditions de sécurité.

Il est important de distinguer les mécanismes suivants :

- **Les jetons de session** : permettent au serveur de retrouver une session utilisateur et son état d'authentification.
- **Les jetons JWT (****_JSON Web Token_****)** : format standardisé permettant de transporter des informations sous forme de revendications (_claims_), éventuellement signées ou chiffrées.
- **Les jetons CSRF (****_Cross-Site Request Forgery_****)** : permettent de protéger certaines requêtes contre les attaques consistant à faire exécuter une action non souhaitée par un utilisateur déjà authentifié.

Ces mécanismes ne sont pas interchangeables. Ils répondent à des besoins différents et peuvent être utilisés conjointement au sein d'une même application.

### 1.1. Objectifs

Cette documentation a pour objectifs de :

- Présenter le fonctionnement des principaux mécanismes de jetons.
- Expliquer leurs avantages, leurs limites et leurs cas d'utilisation.
- Identifier les risques de sécurité associés à leur utilisation.
- Présenter les bonnes pratiques de mise en œuvre.

### 1.2. Terminologie

|Terme|Définition|
|---|---|
|Authentification|Vérification de l'identité d'un utilisateur ou d'un système.|
|Autorisation|Vérification des droits permettant d'effectuer une action ou d'accéder à une ressource.|
|Session|Ensemble d'informations permettant de maintenir un contexte utilisateur entre plusieurs requêtes.|
|Jeton|Valeur transmise ou présentée pour répondre à un besoin d'identification, d'accès ou de vérification.|
|Cookie|Mécanisme du navigateur permettant de conserver et de transmettre des données au serveur.|
|CSRF|Attaque qui exploite l'authentification existante d'un utilisateur pour lui faire exécuter une action non souhaitée.|
|XSS|Vulnérabilité permettant l'exécution de scripts non fiables dans le contexte d'une application web.|

## 2. Les jetons de session

### 2.1. Définition

Un jeton de session est une valeur généralement aléatoire et imprévisible qui permet au serveur de retrouver une session utilisateur.

Dans une architecture classique, les informations de session sont conservées côté serveur. Le client ne possède qu'un identifiant permettant de retrouver ces informations.

Le jeton de session ne doit pas contenir directement des informations sensibles telles que le mot de passe, les permissions détaillées ou les données personnelles de l'utilisateur.

### 2.2. Mode de fonctionnement

Le fonctionnement d'une session repose sur les étapes suivantes :

1. **Authentification** : l'utilisateur transmet ses identifiants au serveur.
2. **Validation** : le serveur vérifie les identifiants.
3. **Création de la session** : le serveur génère un identifiant de session aléatoire et crée les données de session associées.
4. **Transmission au client** : le serveur transmet l'identifiant au navigateur, généralement dans un cookie HTTP.
5. **Requêtes ultérieures** : le navigateur transmet automatiquement le cookie aux requêtes correspondant à son périmètre.
6. **Identification** : le serveur récupère la session associée à l'identifiant et vérifie sa validité.
7. **Fin de session** : lors de la déconnexion ou de l'expiration, le serveur invalide la session.

Les données de session peuvent être conservées en mémoire, dans un système de stockage partagé tel que Redis ou dans une base de données.

### 2.3. Schéma de fonctionnement

mermaid

```
sequenceDiagram
    actor U as Utilisateur
    participant C as Navigateur
    participant S as Serveur
    participant D as Stockage des sessions

    U->>C: Saisie des identifiants
    C->>S: Requête de connexion
    S->>S: Vérification des identifiants
    S->>D: Création de la session
    D-->>S: Identifiant de session
    S-->>C: Réponse Set-Cookie
    C->>S: Requête avec cookie de session
    S->>D: Recherche de la session
    D-->>S: Données de session
    S-->>C: Réponse autorisée
```

### 2.4. Exemple en PHP

L'exemple suivant illustre l'accès à une donnée de session après authentification.

php

```
<?php

session_start();

if (isset($_SESSION['user_id'])) {
    $userId = $_SESSION['user_id'];

    // L'utilisateur dispose d'une session identifiée.
    // Vérifier ensuite ses droits pour l'action demandée.
} else {
    http_response_code(401);
    exit('Authentification requise.');
}
```

La fonction `session_start()` initialise ou reprend une session PHP en fonction de l'identifiant transmis par le client.

**Point de vigilance :** la présence d'un identifiant utilisateur dans la session ne dispense pas de vérifier les autorisations nécessaires pour chaque opération protégée.

Après une authentification réussie, l'identifiant de session doit être renouvelé afin de limiter les risques de fixation de session.

### 2.5. Avantages

- Les données de session restent généralement côté serveur.
- L'invalidation d'une session est simple et peut être immédiate.
- Les modifications des droits peuvent être prises en compte sans attendre l'expiration d'un jeton autonome.
- Le mécanisme est largement pris en charge par les frameworks web.

### 2.6. Inconvénients

- Le serveur doit conserver et gérer l'état des sessions.
- Une architecture distribuée nécessite un stockage partagé ou un mécanisme équivalent.
- Le stockage et la consultation des sessions peuvent représenter un coût opérationnel.
- Le vol d'un identifiant de session peut permettre à un attaquant d'usurper la session de l'utilisateur.

### 2.7. Bonnes pratiques de sécurité

- Utiliser un identifiant de session généré par un générateur cryptographiquement sûr.
- Transmettre les cookies de session uniquement sur HTTPS grâce à l'attribut `Secure`.
- Utiliser l'attribut `HttpOnly` pour empêcher l'accès direct au cookie par JavaScript.
- Configurer `SameSite` de manière adaptée, généralement `Lax` ou `Strict` lorsque l'architecture le permet.
- Renouveler l'identifiant de session après la connexion et lors des changements importants de privilèges.
- Définir une durée maximale de session et, si nécessaire, une durée d'inactivité.
- Invalider la session côté serveur lors de la déconnexion.
- Ne jamais enregistrer les identifiants de session en clair dans les journaux applicatifs.

Exemple d'en-tête de cookie :

http

```
Set-Cookie: __Host-SessionID=<identifiant-aleatoire>; Path=/; Secure; HttpOnly; SameSite=Lax
```

Le préfixe `__Host-` impose notamment l'utilisation de `Secure`, l'absence d'attribut `Domain` et `Path=/` dans les navigateurs qui prennent en charge ces règles.

Référence : [OWASP — Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).

## 3. Les jetons JWT (JSON Web Token)

### 3.1. Définition

Un JWT (_JSON Web Token_) est un format normalisé de jeton défini par la RFC 7519.

Il permet de transporter des informations structurées sous forme de revendications, appelées _claims_. Selon son type, le jeton peut être signé, chiffré ou les deux.

Un JWT signé peut être vérifié sans que le serveur ait besoin de consulter une session stockée pour chaque requête. Ce fonctionnement est souvent qualifié de _stateless_.

Cependant, **l'utilisation d'un JWT ne garantit pas à elle seule une architecture sans état**. Une application peut conserver une liste de révocation, gérer des jetons de renouvellement ou consulter une base de données pour vérifier les droits d'un utilisateur.

### 3.2. Structure d'un JWT signé

Un JWT signé au format compact comporte généralement trois parties séparées par un point (`.`) :

text

```
header.payload.signature
```

#### a. Header (en-tête)

L'en-tête indique notamment le type de jeton et l'algorithme utilisé pour la signature.

Exemple :


```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

`HS256` désigne un mécanisme de signature fondé sur HMAC et SHA-256.

#### b. Payload (données)

Le contenu regroupe les revendications du jeton.

Exemple :


```json
{
  "sub": "12345",
  "iss": "https://auth.example.com",
  "aud": "api.example.com",
  "iat": 1790000000,
  "exp": 1790000900,
  "roles": ["user"]
}
```

Les champs courants sont notamment :

|Claim|Signification|
|---|---|
|`iss`|Émetteur du jeton (_issuer_).|
|`sub`|Sujet du jeton, généralement l'identifiant de l'utilisateur.|
|`aud`|Destinataire auquel le jeton est destiné.|
|`iat`|Date d'émission du jeton.|
|`nbf`|Date avant laquelle le jeton ne doit pas être accepté.|
|`exp`|Date d'expiration du jeton.|
|`jti`|Identifiant unique du jeton, notamment utile pour certains mécanismes de révocation.|

Les noms de revendications personnalisées, tels que `roles`, dépendent de l'application.

#### c. Signature

La signature permet au destinataire de vérifier l'intégrité du jeton et, selon l'algorithme, son origine.

Avec un algorithme tel que `HS256`, la signature repose sur un secret partagé entre l'émetteur et le vérificateur.

Avec un algorithme asymétrique, l'émetteur signe avec une clé privée et les destinataires vérifient la signature à l'aide de la clé publique correspondante.

### 3.3. Encodage, signature et chiffrement

Il est essentiel de distinguer ces trois notions :

- **Encodage** : transforme les données en une représentation adaptée au transport. L'encodage Base64URL ne protège pas la confidentialité.
- **Signature** : permet de détecter les modifications et de vérifier l'authenticité du jeton selon la clé utilisée.
- **Chiffrement** : protège la confidentialité du contenu contre les personnes ne disposant pas de la clé de déchiffrement.

Un JWT signé classique est généralement lisible par toute personne qui le possède. Il ne faut donc pas y intégrer de mots de passe, de secrets ou d'autres informations confidentielles.

### 3.4. Mode de fonctionnement

Le cycle d'utilisation d'un JWT peut être résumé ainsi :

1. Le client transmet ses identifiants au serveur d'authentification.
2. Le serveur vérifie les identifiants.
3. Le serveur génère un jeton signé contenant les revendications nécessaires.
4. Le jeton est transmis au client.
5. Le client présente le jeton lors des requêtes protégées, généralement dans l'en-tête HTTP `Authorization`.
6. Le serveur vérifie la signature, les dates de validité, l'émetteur et le destinataire attendus.
7. Le serveur vérifie les autorisations nécessaires à l'opération demandée.
8. Le jeton est rejeté s'il est invalide, expiré ou inadapté au contexte.

Exemple d'en-tête HTTP :

http

```
Authorization: Bearer <jwt>
```

### 3.5. Exemple de validation

L'exemple suivant illustre la logique générale attendue. Il ne constitue pas une implémentation complète.

```
fonction traiterRequete(requete):
    jeton = extraireBearer(requete.headers["Authorization"])

    si jeton est absent:
        retourner 401

    revendications = verifierEtDecoderJWT(
        jeton,
        algorithmesAutorises,
        emetteurAttendu,
        destinataireAttendu
    )

    si la signature ou les dates sont invalides:
        retourner 401

    utilisateur = chargerUtilisateur(revendications.sub)

    si utilisateur est absent ou inactif:
        retourner 401

    si utilisateur non autorisé pour l'action:
        retourner 403

    exécuterAction()
```

La fonction de validation doit utiliser une bibliothèque JWT maintenue et une configuration explicite des algorithmes autorisés. Il ne suffit pas de décoder le contenu du jeton pour considérer celui-ci comme fiable.

### 3.6. Avantages

- La vérification d'un jeton signé peut être réalisée localement par plusieurs services.
- Le mécanisme facilite les architectures distribuées et certaines API.
- Le serveur n'a pas nécessairement besoin de consulter une session pour chaque requête.
- Les revendications standardisées facilitent l'interopérabilité lorsqu'elles sont utilisées conformément au protocole concerné.

### 3.7. Inconvénients

- Un jeton volé peut être utilisé par un attaquant jusqu'à son expiration ou sa révocation effective.
- La révocation immédiate nécessite un mécanisme complémentaire ou une durée de validité réduite.
- Les revendications peuvent devenir obsolètes si les droits de l'utilisateur changent.
- Une mauvaise configuration cryptographique peut compromettre la sécurité.
- La taille des jetons peut augmenter la taille des requêtes HTTP.
- Le stockage côté client expose le jeton aux risques liés à son environnement de stockage.

### 3.8. Bonnes pratiques de sécurité

- Utiliser une bibliothèque reconnue et maintenue.
- Définir explicitement les algorithmes autorisés et rejeter les algorithmes inattendus.
- Vérifier la signature et les revendications pertinentes, notamment `iss`, `aud`, `exp` et `nbf` lorsqu'elles sont présentes et requises par le protocole.
- Définir des durées de validité adaptées au niveau de risque.
- Ne pas considérer les revendications du jeton comme fiables avant validation cryptographique.
- Éviter d'y placer des informations sensibles.
- Prévoir une stratégie de renouvellement, de rotation des clés et, si nécessaire, de révocation.
- Ne jamais accepter un jeton simplement parce que son format semble correct.
- Éviter de stocker les jetons dans `localStorage` lorsque des scripts malveillants pourraient y accéder en cas de vulnérabilité XSS.

Le choix du stockage doit être adapté à l'application. Un cookie `HttpOnly` réduit l'accès direct au jeton depuis JavaScript, mais son envoi automatique par le navigateur impose également de traiter les risques CSRF lorsque cela est pertinent.

Référence : RFC 7519 — JSON Web Token et RFC 8725 — JSON Web Token Best Current Practices.

## 4. Les jetons CSRF (Cross-Site Request Forgery)

### 4.1. Définition

Un jeton CSRF est une valeur utilisée pour vérifier qu'une requête sensible provient d'un contexte autorisé et n'a pas simplement été déclenchée par un site tiers exploitant la session d'un utilisateur.

Une attaque CSRF peut survenir lorsqu'un navigateur est déjà authentifié auprès d'une application et qu'un site malveillant tente de lui faire envoyer une requête vers cette application.

Ce risque est particulièrement important lorsque l'authentification repose sur des cookies automatiquement transmis par le navigateur.

Le jeton CSRF ne remplace pas l'authentification. Il complète les mécanismes de sécurité en protégeant les opérations qui doivent être à l'abri de ce type de requête forgée.

### 4.2. Mode de fonctionnement

Dans le modèle classique dit _Synchronizer Token Pattern_ :

1. Le serveur crée une session utilisateur.
2. Le serveur génère un jeton CSRF imprévisible et l'associe à cette session.
3. Le jeton est transmis à la page web, par exemple dans un formulaire HTML ou une réponse destinée au client JavaScript.
4. Lors d'une opération protégée, le client renvoie le jeton dans un champ de formulaire ou un en-tête HTTP personnalisé.
5. Le serveur compare le jeton reçu avec celui associé à la session.
6. Si le jeton est absent ou invalide, le serveur rejette la requête.

Dans ce modèle, le jeton est généralement conservé côté serveur. Il peut être valable pendant toute la session ou être renouvelé plus fréquemment.

**Attention :** placer le jeton dans un cookie ne suffit pas à protéger contre le CSRF si le serveur ne vérifie que ce cookie. Le navigateur peut transmettre automatiquement un cookie à la requête forgée. Avec le modèle synchronisé, le jeton doit être renvoyé séparément, par exemple dans un en-tête ou un champ de formulaire.

### 4.3. Schéma de fonctionnement

mermaid

```
sequenceDiagram
    actor U as Utilisateur
    participant C as Navigateur
    participant S as Serveur

    U->>C: Accès à une page protégée
    C->>S: Requête avec cookie de session
    S-->>C: Page et jeton CSRF
    U->>C: Soumission du formulaire
    C->>S: Requête avec cookie et jeton CSRF
    S->>S: Vérification de la session
    S->>S: Vérification du jeton CSRF
    alt Jeton valide
        S-->>C: Action autorisée
    else Jeton absent ou invalide
        S-->>C: Requête rejetée
    end
```

### 4.4. Exemple HTML

Le serveur fournit un jeton CSRF à la page, puis vérifie ce jeton lors de la réception du formulaire.

```html
<form method="POST" action="/profil/modifier">
    <input
        type="hidden"
        name="csrf_token"
        value="{{ csrf_token }}"
    >

    <label for="display_name">Nom affiché</label>
    <input
        id="display_name"
        name="display_name"
        type="text"
    >

    <button type="submit">Enregistrer</button>
</form>
```

`{{ csrf_token }}` représente ici un emplacement à remplacer par le mécanisme de rendu sécurisé du framework utilisé.

Le serveur doit valider le jeton avant d'effectuer la modification. Le champ caché ne constitue pas, à lui seul, une protection : la vérification côté serveur est indispensable.

### 4.5. Utilisation avec JavaScript et AJAX

Pour une application qui utilise `fetch` ou un autre client HTTP, le jeton peut être transmis dans un en-tête personnalisé.

Exemple :

```
<meta name="csrf-token" content="{{ csrf_token }}">
```


```javascript
const csrfToken = document
    .querySelector('meta[name="csrf-token"]')
    .getAttribute('content');

fetch('/api/profil/modifier', {
    method: 'POST',
    credentials: 'same-origin',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrfToken
    },
    body: JSON.stringify({
        displayName: 'Nouvelle valeur'
    })
});
```

Dans cet exemple, le navigateur transmet les cookies de même origine grâce à `credentials: 'same-origin'`, tandis que le code JavaScript ajoute explicitement le jeton CSRF.

Le serveur doit vérifier le jeton `X-CSRF-Token` avant de traiter l'opération. La simple présence de l'en-tête ne suffit pas.

Le nom de l'en-tête peut varier selon le framework ou les conventions de l'application.
### 4.6. Avantages

- Réduit les risques d'actions forgées depuis un site tiers.
- Complète la protection offerte par les cookies de session.
- Peut être intégré aux formulaires HTML et aux appels AJAX.
- Peut être mis en œuvre sans rendre chaque jeton strictement limité à une seule requête.

### 4.7. Inconvénients et limites

- Nécessite une génération et une validation correctes des jetons.
- Dans le modèle synchronisé, le jeton doit être associé à la session et conservé côté serveur.
- Une mauvaise gestion du renouvellement peut rendre des formulaires ouverts invalides.
- Une faille XSS peut permettre à un attaquant d'exécuter du JavaScript dans le contexte de l'application et de récupérer le jeton.
- Une mauvaise configuration CORS ou une exposition du jeton peut affaiblir la protection.

Il existe également d'autres stratégies, comme le modèle _Double Submit Cookie_ et certaines protections fondées sur la vérification de l'origine. Leur pertinence dépend de l'architecture de l'application.

### 4.8. Bonnes pratiques de sécurité

- Utiliser un générateur cryptographiquement sûr pour créer les jetons.
- Associer le jeton à la session ou utiliser une stratégie alternative reconnue et correctement implémentée.
- Vérifier le jeton côté serveur pour chaque opération nécessitant une protection CSRF.
- Ne jamais placer le jeton dans une URL, où il pourrait être exposé dans les journaux, l'historique ou l'en-tête `Referer`.
- Configurer les cookies de session avec un attribut `SameSite` adapté.
- Vérifier l'origine des requêtes sensibles lorsque cela est pertinent.
- Ne pas utiliser de requêtes `GET` pour effectuer des opérations qui modifient l'état de l'application.
- Utiliser les mécanismes CSRF fournis par le framework lorsque ceux-ci sont disponibles.
- Considérer la protection CSRF comme complémentaire à la prévention des failles XSS.

Référence : [OWASP — Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).

## 5. Comparaison des mécanismes

Les trois mécanismes présentés ne répondent pas au même besoin.

|Critère|Jeton de session|JWT|Jeton CSRF|
|---|---|---|---|
|Objectif principal|Maintenir une session authentifiée|Transporter des revendications vérifiables|Protéger contre certaines requêtes forgées|
|Stockage habituel côté serveur|État de session conservé|Pas nécessairement d'état par jeton|Jeton associé à la session dans le modèle synchronisé|
|Validation|Recherche et vérification de la session|Vérification cryptographique et des revendications|Vérification du jeton et du contexte de la requête|
|Invalidation immédiate|Généralement simple|Nécessite un mécanisme complémentaire ou une expiration adaptée|Dépend de la stratégie et de la durée de validité|
|Utilisation typique|Application web avec cookies|API et architectures distribuées|Formulaires et opérations sensibles utilisant notamment des cookies|
|Risque principal|Vol ou fixation de session|Vol ou mauvaise validation du jeton|Requête forgée exploitant l'authentification du navigateur|

Il ne faut pas conclure qu'un JWT est systématiquement plus performant ou plus sécurisé qu'une session. Le choix dépend des besoins fonctionnels, de l'architecture, du cycle de vie des identifiants et des exigences de sécurité.

## 6. Autres notions importantes : access tokens et refresh tokens

Les notions d'_access token_ et de _refresh token_ sont particulièrement courantes dans les architectures utilisant OAuth 2.0.

Elles ne désignent pas un format particulier : un access token peut être un JWT ou un jeton opaque, dont le contenu n'est pas directement interprétable par le client.

### 6.1. Access token

L'access token permet à un client d'accéder à une ressource protégée dans les limites des droits qui lui ont été accordés.

Il doit être transmis uniquement aux destinataires autorisés et être validé conformément au protocole utilisé.

### 6.2. Refresh token

Le refresh token permet, dans les flux qui le prévoient, d'obtenir un nouvel access token sans demander systématiquement à l'utilisateur de saisir à nouveau ses identifiants.

Il nécessite une protection renforcée, car sa compromission peut permettre d'obtenir de nouveaux access tokens.

Les mesures de sécurité comprennent notamment :

- Le stockage sécurisé du refresh token.
- La rotation des refresh tokens lorsque le protocole et le fournisseur le permettent.
- La détection de la réutilisation d'un ancien refresh token dans les mécanismes qui la prennent en charge.
- La révocation des jetons lors d'événements de sécurité ou de déconnexion lorsque le système le permet.

Il convient de ne pas confondre un access token, un refresh token, un JWT et un jeton de session : ces termes décrivent respectivement un rôle dans le contrôle d'accès, un rôle dans le renouvellement, un format de jeton et un mécanisme de gestion de session.

## 7. Recommandations générales

Pour concevoir une application web sécurisée, les mesures suivantes doivent être considérées.

### 7.1. Transport et stockage

- Utiliser HTTPS sur l'ensemble des échanges authentifiés.
- Protéger les cookies d'authentification avec des attributs adaptés, notamment `Secure`, `HttpOnly` et `SameSite`.
- Éviter de stocker les jetons d'authentification dans des emplacements accessibles à tout JavaScript de l'application, lorsque cela peut être évité.
- Ne jamais transmettre de jeton d'authentification dans une URL.

### 7.2. Cycle de vie

- Définir des durées de validité adaptées au risque.
- Invalider les sessions côté serveur lors de la déconnexion.
- Prévoir le renouvellement et la révocation des jetons lorsque cela est nécessaire.
- Renouveler l'identifiant de session après authentification.
- Réévaluer les droits pour les opérations sensibles.

### 7.3. Validation et autorisation

- Vérifier systématiquement l'authenticité des jetons avant de les utiliser.
- Appliquer le principe du moindre privilège.
- Contrôler les autorisations côté serveur pour chaque opération protégée.
- Ne jamais faire confiance à une donnée reçue du client uniquement parce qu'elle figure dans un jeton.
- Ne pas confondre une requête authentifiée avec une requête autorisée.

### 7.4. Journalisation et supervision

- Ne pas enregistrer les mots de passe, les cookies de session ou les jetons complets dans les journaux.
- Enregistrer les événements pertinents : échecs d'authentification, expirations, révocations et tentatives d'accès non autorisées.
- Limiter l'accès aux journaux et protéger les données sensibles qui y figurent.
- Mettre en place des alertes adaptées aux événements de sécurité.

## 8. Conclusion

Les jetons de session, les JWT et les jetons CSRF constituent des mécanismes complémentaires, mais leurs objectifs sont différents.

Les jetons de session conviennent notamment aux applications web qui souhaitent conserver un état d'authentification contrôlé côté serveur. Les JWT sont utiles lorsque le transport de revendications vérifiables facilite les échanges entre services. Les jetons CSRF protègent les opérations sensibles contre certaines attaques qui exploitent l'envoi automatique des cookies par le navigateur.

Le choix d'une solution doit reposer sur les besoins de l'application, son architecture et les risques associés. La sécurité ne dépend pas uniquement du type de jeton utilisé, mais aussi de sa génération, de son stockage, de sa transmission, de sa validation, de son expiration et de sa révocation.

## 9. Références

- [OWASP — Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP — Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- RFC 7519 — JSON Web Token
- RFC 8725 — JSON Web Token Best Current Practices
- OWASP — Cross Site Scripting Prevention Cheat Sheet