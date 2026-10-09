
Les scripts PHP placés dans les sous-répertoires `ajax/` ne doivent pas pouvoir être appelés librement depuis l'extérieur de l'application.

Par exemple, un script comme :

```text
ajax/getbyname.php
```

doit être appelé uniquement depuis une page de notre application et non directement par une requête externe.

Pour cela, l'application utilise un **jeton d'accès AJAX**.

Ce jeton est généré côté serveur, associé à la session de l'utilisateur et transmis automatiquement lors des appels AJAX.

> Le mécanisme utilisé est similaire à celui d'un jeton CSRF, mais dans notre application nous l'utilisons principalement pour contrôler l'accès aux scripts AJAX.
## 1. Principe

Le fonctionnement est simple :

```text
Contrôleur PHP
      ↓
génération du jeton
      ↓
page HTML
      ↓
appelAjax()
      ↓
ajout automatique du jeton à la requête
      ↓
script PHP dans ajax/
      ↓
vérification du jeton
      ↓
 ┌───────────────┐
 │               │
valide         invalide
 │               │
 ↓               ↓
exécution      refus
```

Le jeton permet donc au script AJAX de vérifier que la requête provient bien d'une page de notre application.

## 2. Générer le jeton

Le contrôleur indique que la page doit utiliser un jeton :

```php
$page->avecJeton();
```

Par exemple :

```php
$page = new Page();

$page->setTitre("Recherche d'un coureur")
    ->avecJeton()
    ->addComposant("autocomplete")
    ->setDonnee('lesCoureurs', Coureur::getListe())
    ->afficher();
```

Le système génère alors un jeton et l'intègre dans la page HTML.

Le jeton peut par exemple être placé dans une balise `meta` :

```html
<meta name="csrf-token" content="abc123xyz...">
```

Le développeur n'a pas besoin de gérer directement cette balise.

## 3. Transmission du jeton lors d'un appel AJAX

Les appels AJAX de l'application doivent utiliser la fonction :

```javascript
appelAjax()
```

Par exemple :

```javascript
appelAjax({
    url: "ajax/getbyname.php",
    method: "GET",
    data: {
        search: query
    },
    dataType: "json"
});
```

La fonction `appelAjax()` récupère automatiquement le jeton présent dans la page et l'ajoute à la requête HTTP.

Le développeur n'a donc pas besoin d'ajouter manuellement le jeton à chaque appel.

La requête envoyée au serveur contient notamment :

```text
X-CSRF-Token: abc123xyz...
```

## 4. Vérification dans le script AJAX

Lorsqu'un script situé dans `ajax/` reçoit une requête, il vérifie le jeton transmis en appelant ma méthode Requete::exigerPost()

Par exemple :

```php
<?php  
use ClasseMetier\Projet;  
  
use ClasseTechnique\Requete;  
use ClasseTechnique\ReponseJson;  
  
/** @noinspection PhpIncludeInspection */  
require $_SERVER['DOCUMENT_ROOT'] . "/../bootstrap/bootstrap.php";  
  
// Vérification d'un appel AJAX par la méthode POST sécurisé par un jeton  
Requete::exigerPost();  
  
// récupérer en vérifiant la présence et le type des paramètres attendus  
$idProjet = Requete::postInt('idProjet');  
  
  
// Si le projet n'existe pas, on envoie une erreur  
if (!Projet::getById($idProjet)) {  
    ReponseJson::envoyerLesErreurs(['global' => "Ce projet n'existe pas."]);  
}  
  
// récupération des compétences du projet et envoi de la réponse au format json  
ReponseJson::envoyerLesDonnees(Projet::getLesCompetences($idProjet));
```

La méthode exisgerPost() effectue trois vérifications :
+ l'entête HTTP_X_REQUESTED_WITH doit être transmise avec la valeur 'xmlhttprequest' ce qui est la signature d'une appel Ajax
+ la méthode d'envoie des données doit correspondre à la méthode POST
+ le jeton doit être transmis dans l'entête 'HTTP_X_CSRF_TOKEN' et doit correspondre à celui conservé côté serveur dans la variable de session $_SESSION['csrf_token']


```text
Jeton reçu : $_SERVER['HTTP_X_CSRF_TOKEN']
     ↓
Comparaison avec le jeton de la session : $_SESSION['csrf_token']
     ↓
 ┌───────────────┐
 │               │
Identique      Différent
 │               │
 ↓               ↓
Autorisé        Refusé
```

Si le jeton est valide, le script peut poursuivre son traitement.

S'il est absent ou incorrect, la requête est refusée.

Par exemple :

```php
http_response_code(403);
exit;
```

## 5. Pourquoi cette protection ?

Sans cette vérification, une personne pourrait tenter d'appeler directement un script AJAX :

```text
https://monapplication.fr/module/ajax/getbyname.php
```

Le script pourrait alors être utilisé indépendamment de l'application.

Avec le jeton, le script vérifie que la requête possède bien le jeton attendu.

```text
Requête vers ajax/getbyname.php
             ↓
       Jeton présent ?
          /       \
        NON       OUI
        ↓          ↓
      Refus     Jeton valide ?
                   /     \
                 NON      OUI
                 ↓         ↓
               Refus    Exécution
```

## 6. Le rôle de `appelAjax()`

Pour les développeurs JavaScript, la règle est simple :

> **Tous les appels vers un script situé dans un répertoire `ajax/` doivent être réalisés avec `appelAjax()`.**

Il ne faut pas utiliser directement :

```javascript
fetch(...)
```

pour appeler ces scripts.

La fonction `appelAjax()` centralise en effet les mécanismes nécessaires aux appels AJAX de l'application, notamment la transmission du jeton d'accès.

Le développeur peut donc simplement écrire :

```javascript
appelAjax({
    url: "ajax/getbyname.php",
    method: "GET",
    data: {
        search: query
    },
    dataType: "json"
});
```

et laisser `appelAjax()` gérer la partie sécurité.

## 7. À retenir

Le mécanisme repose sur trois éléments :

|Élément|Rôle|
|---|---|
|`avecJeton()`|Demande au serveur de protéger la page avec un jeton.|
|`appelAjax()`|Transmet automatiquement le jeton lors des appels AJAX.|
|Script PHP dans `ajax/`|Vérifie le jeton avant d'exécuter le traitement.|

Le principe général est donc :

```text
1. Le contrôleur génère un jeton
             ↓
2. Le jeton est transmis à la page
             ↓
3. appelAjax() le transmet automatiquement
             ↓
4. Le script AJAX vérifie le jeton
             ↓
5. Jeton valide → traitement
   Jeton invalide → accès refusé
```

### Règle pour les développeurs

Lorsqu'un nouveau script PHP est créé dans un répertoire `ajax/` :

- le script doit vérifier le jeton d'accès ;
- les appels JavaScript doivent utiliser `appelAjax()` ;
- le contrôleur de la page doit activer la protection avec `avecJeton()`.

Le développeur n'a ainsi pas à gérer lui-même la génération ou la transmission du jeton : **le framework s'en charge automatiquement.**