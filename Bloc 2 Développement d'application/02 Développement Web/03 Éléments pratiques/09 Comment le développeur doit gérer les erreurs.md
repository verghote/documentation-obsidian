L'application dispose de plusieurs mécanismes pour gérer les erreurs.

Le choix du mécanisme dépend principalement de **la manière dont le script PHP est appelé** et du **type de traitement attendu**.

Le développeur ne doit donc pas choisir une solution uniquement en fonction du nom de l'erreur, mais d'abord en fonction du **contexte de restitution**.

# 1. Quelle solution utiliser ?

Avant d'écrire le traitement d'une erreur, déterminer comment le script est appelé.

| Situation                                                  | Solution à utiliser de préférence | Pourquoi ?                                                  |
| ---------------------------------------------------------- | --------------------------------- | ----------------------------------------------------------- |
| Le script est appelé directement par le navigateur         | `InterfaceSystem`                 | Le script doit retourner une page HTML                      |
| Le script est exclusivement appelé par `appelAjax()`       | `ReponseJson`                     | `appelAjax()` attend une réponse JSON                       |
| Le même traitement peut être appelé directement ou en AJAX | `UserException`                   | `Erreur` adapte automatiquement la restitution              |
| Une erreur technique imprévue se produit                   | Laisser l'exception remonter      | `Erreur` assure la journalisation et la restitution adaptée |

## Règle pratique

> **Navigation directe → `InterfaceSystem`**  
> **AJAX → `ReponseJson`**  
> **Plusieurs contextes d'appel → `UserException`**  
> **Erreur technique imprévue → laisser l'exception remonter**

Cette règle constitue le choix par défaut.

Les autres possibilités restent techniquement disponibles, mais elles ne sont pas nécessairement les plus appropriées.

# 2. Script appelé directement par le navigateur

Lorsqu'un script PHP est appelé directement par une navigation normale, il peut afficher directement une interface système.

Dans ce cas, utiliser :

```php
use ClasseTechnique\InterfaceSystem;
```

La classe `InterfaceSystem` permet d'afficher une page intégrée à la charte graphique de l'application.

Elle accepte quatre types de messages :

```text
erreur
avertissement
information
maintenance
```

## Exemple

```php
$ligne = Annonce::getById($id);

if (!$ligne) {

    $titre = "Annonce inexistante";
    $message = "L'annonce $id n'existe pas.";
    $type = "avertissement";

    $interface = new InterfaceSystem($titre, $message, $type);

    $interface->afficher();
}
```

La page système est alors générée directement par le script.

## Pourquoi utiliser `InterfaceSystem` ?

Cette solution est à privilégier lorsque le script **sait qu'il est exclusivement appelé par le navigateur**.

Il est alors inutile de :

- déterminer si l'appel est AJAX ;
- construire une réponse JSON ;
- stocker le message dans la session ;
- rediriger vers `/erreur`.

Le script connaît déjà le mode de restitution attendu :

**une page HTML.**

`InterfaceSystem` présente également l'avantage de permettre au développeur de choisir :

- le titre de l'interface ;
- le type du message ;
- donc l'apparence associée au type de situation.

## Peut-on utiliser `UserException` à la place ?

Oui.

Il est techniquement possible d'écrire :

```php
if (!$ligne) {
    throw new UserException("Cette annonce n'existe pas.");
}
```

La classe `Erreur` interceptera alors l'exception et déterminera le contexte d'appel.

Cependant, lorsque le script est exclusivement destiné à une navigation normale, cette solution est moins explicite.

Elle déclenche un mécanisme de détection du contexte qui n'est pas nécessaire.

De plus, `UserException` ne permet pas au script de préciser directement :

- le titre de l'interface ;
- le type du message (`erreur`, `avertissement`, etc.).

Pour un script exclusivement destiné au navigateur, **`InterfaceSystem` est donc préférable**.

# 3. Script appelé en AJAX

Les scripts placés dans le répertoire `ajax` sont appelés par la fonction JavaScript :

```javascript
appelAjax()
```

Ils ne doivent donc pas retourner une page HTML.

Ils doivent retourner une **réponse JSON**.

Pour cela, utiliser les méthodes de la classe technique `ReponseJson`.

La fonction `appelAjax()` interprète automatiquement la réponse et assure l'affichage approprié côté navigateur.

## Retourner des données

Lorsque le traitement réussit et doit retourner des données au JavaScript :

```php
ReponseJson::envoyerLesDonnees(Coureur::getByNomPrenom($search));
```

## Retourner un message de succès

Lorsque le traitement réussit mais qu'il n'y a pas de données particulières à retourner :

```php
ReponseJson::envoyerMessage('Votre mot de passe a été réinitialisé avec succès.');
```

## Retourner une erreur globale

Lorsque l'erreur concerne l'ensemble de la demande :

```php
ReponseJson::envoyerErreur('La requête est invalide.');
```

Par défaut, `appelAjax()` affiche le message dans :

```html
<div id="msg">
```

si cette zone existe.

À défaut, le message est affiché dans une fenêtre de message.

## Retourner des erreurs de validation

Lorsque les erreurs concernent un ou plusieurs champs :

```php
ReponseJson::envoyerLesErreurs([
    'login' => 'Le login est obligatoire.',
    'password' => 'Le mot de passe est incorrect.'
]);
```

La clé correspond à l'identifiant du champ concerné.

`appelAjax()` utilise ces clés pour afficher automatiquement les messages au bon endroit.

Une erreur globale peut également être ajoutée :

```php
ReponseJson::envoyerLesErreurs([
    'login' => 'Le login est obligatoire.',
    'global' => 'Impossible de traiter votre demande.'
]);
```

La clé `global` indique que le message ne concerne pas un champ particulier.
## Construire une réponse particulière

Lorsque la réponse doit contenir une structure spécifique :

```php
ReponseJson::envoyer([
    'url' => $url,
    'message' => $message
]);
```

Cette méthode est utilisée lorsque les méthodes spécialisées précédentes ne correspondent pas au besoin.

## Les principales méthodes de `ReponseJson`

| Besoin                                           | Méthode               |
| ------------------------------------------------ | --------------------- |
| Retourner des données                            | `envoyerLesDonnees()` |
| Retourner un message de succès                   | `envoyerMessage()`    |
| Retourner une erreur globale                     | `envoyerErreur()`     |
| Retourner une ou plusieurs erreurs de validation | `envoyerLesErreurs()` |
| Construire une réponse particulière              | `envoyer()`           |

# 4. Utiliser `UserException` dans un contrôleur AJAX

Même dans un contrôleur exclusivement AJAX, il est techniquement possible d'utiliser `UserException`.

Par exemple :

```php
$idProjet = Requete::postInt('idProjet');

if (!Projet::existe($idProjet)) {
    throw new UserException("Ce projet n'existe pas.");
}
```

La classe `Erreur` détectera que l'appel est AJAX et produira automatiquement la réponse JSON appropriée.

On aurait également pu écrire :

```php
$idProjet = Requete::postInt('idProjet');

if (!Projet::existe($idProjet)) {
    ReponseJson::envoyerLesErreurs(['global' => "Ce projet n'existe pas."]);
}
```

Les deux solutions fonctionnent.

Cependant, pour un contrôleur exclusivement AJAX, `ReponseJson` est généralement préférable car elle indique immédiatement au lecteur du code que le contrôleur retourne une réponse JSON.

`UserException` devient particulièrement intéressante lorsque le même traitement doit pouvoir fonctionner dans plusieurs contextes.

# 5. Utiliser `UserException`

`UserException` permet de signaler une **erreur destinée à l'utilisateur** sans que le traitement ait à connaître le mode de restitution final.

Exemple :

```php
throw new UserException('Le lien de connexion est invalide ou a expiré.');
```

La classe `Erreur` intercepte cette exception et détermine le contexte d'appel.

Elle peut alors :

- produire une réponse JSON si l'appel est AJAX ;
- enregistrer le message en session et rediriger vers `/erreur` lors d'une navigation normale.

## Code HTTP

Une `UserException` peut être associée à un code HTTP :

```php
throw new UserException("Cette ressource n'existe pas.", 404);
```

Par exemple :

```text
400 → requête incorrecte
401 → authentification nécessaire
403 → accès interdit
404 → ressource inexistante
409 → conflit
```

Le code HTTP permet d'associer la réponse à la situation rencontrée.

## Pourquoi choisir `UserException` ?

Utiliser `UserException` lorsque **le traitement ne doit pas connaître le contexte dans lequel il est exécuté**.

Par exemple :

```text
                Traitement
                    │
             ┌──────┴──────┐
             │             │
        Navigation       AJAX
             │             │
             ▼             ▼
          HTML           JSON
```

Le traitement peut simplement effectuer :

```php
throw new UserException('Le projet demandé n\'existe pas.', 404);
```

Il n'a pas besoin de savoir si le résultat final doit être :

- une page HTML ;
- une réponse JSON.

C'est la classe `Erreur` qui s'en charge.

# 6. Ne pas utiliser `UserException` systématiquement

`UserException` peut techniquement être utilisée dans les différents contextes présentés précédemment.

Cela ne signifie pas qu'elle doit être utilisée systématiquement.

Lorsque le contexte est connu, il est généralement préférable d'utiliser directement le mécanisme correspondant.

Pour un script exclusivement destiné au navigateur :

```php
InterfaceSystem
```

est plus explicite.

Pour un contrôleur exclusivement AJAX :

```php
ReponseJson
```

est plus explicite.

`UserException` est surtout intéressante lorsque **le contexte d'appel peut varier** ou lorsque le traitement doit rester indépendant de son mode de restitution.

On peut donc retenir :

```text
Contexte connu
     │
     ├── Navigateur → InterfaceSystem
     │
     └── AJAX       → ReponseJson

Contexte variable
     │
     └── UserException
```

# 7. Erreur utilisateur ou erreur technique ?

Il est important de distinguer deux catégories de situations.

## Erreur destinée à l'utilisateur

Une erreur utilisateur survient lorsqu'une action demandée ne peut pas être réalisée dans les conditions actuelles.

Exemples :

```text
Le projet demandé n'existe pas.
Le mot de passe est incorrect.
Le lien de connexion a expiré.
Le fichier demandé est introuvable.
```

Ce type de situation peut être présenté à l'utilisateur.

Selon le contexte, on utilisera :

```text
InterfaceSystem
ReponseJson
UserException
```

## Erreur technique

Une erreur technique survient lorsque l'application rencontre un problème qui n'était pas prévu par le fonctionnement normal de l'application.

Exemples :

```text
Erreur de connexion à la base de données.
Erreur SQL.
Fichier de configuration illisible.
Classe inexistante.
Erreur de programmation.
```

Dans ce cas, **ne pas transformer systématiquement l'exception en message utilisateur**.

Il faut généralement laisser l'exception remonter.

# 8. Cas des erreurs techniques imprévues

Lorsqu'une erreur technique imprévue se produit, il ne faut généralement rien faire.

Exemple :

```php
$projet = Projet::getById($idProjet);
```

Si une exception technique survient, ne pas écrire :

```php
try {
    $projet = Projet::getById($idProjet);
} catch (Exception $e) {
    echo $e->getMessage();
}
```

Il ne faut surtout pas afficher directement les détails techniques d'une exception.

La classe `Erreur` prend en charge l'erreur globale.

Elle assure notamment :

- la journalisation des informations techniques ;
- le masquage des détails internes ;
- la production d'un message compréhensible par l'utilisateur ;
- l'adaptation de la réponse au contexte d'appel.

Le développeur doit donc éviter de capturer une exception technique uniquement pour afficher son message.

# 9. Quelles erreurs sont journalisées ?

La classe `Erreur` journalise **toute exception technique non interceptée** à l'exception des 'UserException'

Une `UserException` est considérée comme une erreur applicative volontairement destinée à l'utilisateur. Elle n'est donc pas journalisée.

Le message est directement utilisé pour construire la réponse utilisateur.

Toute `PDOException` est journalisée.

Cela concerne notamment :

- violation d'une contrainte `UNIQUE` ;
- violation d'une contrainte `NOT NULL` ;
- violation d'une contrainte `FOREIGN KEY` ;
- valeur trop longue ;
- valeur incompatible avec le type attendu ;
- violation d'une contrainte `CHECK` ;
- erreur provenant d'un trigger SQL avec `SIGNAL` ;
- erreur SQL non prévue ou non reconnue.

Après journalisation, `Erreur` essaie de traduire certaines erreurs SQL en message compréhensible.

Par exemple :

```text
1062 → Enregistrement déjà existant.
1048 → Une information obligatoire est manquante.
1406 → Une information est trop longue.
1366 → Format de donnée invalide.
1452 → Donnée invalide (référence inexistante).
1451 → Suppression impossible : donnée utilisée.
3819 / 4025 → règle CHECK non respectée.
```

Une erreur SQL non reconnue reçoit le message générique.

## Les autres `Throwable`

Toute autre classe qui implémente `Throwable` et qui n'est ni une `UserException` ni une `PDOException` est journalisée.

Cela concerne notamment :

```text
Exception
RuntimeException
Error
TypeError
```

ainsi que les autres types de `Throwable`.

Le principe est donc :

|Type|Journalisée ?|Message utilisateur|
|---|---|---|
|`UserException`|**Non**|Message réel|
|`PDOException`|**Oui**|Message SQL traduit ou message générique|
|Autre `Throwable`|**Oui**|Message générique|

## Informations enregistrées dans le journal

Pour une erreur technique, `Erreur` enregistre notamment :

```text
[ClasseException]
message
code
fichier
ligne
```

Cela permet au développeur de retrouver l'origine du problème sans exposer ces informations au navigateur.

# 10. Exemple complet d'un contrôleur AJAX

```php
declare(strict_types=1);

use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;
use ClasseMetier\Coureur;

require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/bootstrap.php';

Requete::exigerGet();

$search = Requete::getString('search');

if (trim($search) === '') {

    ReponseJson::envoyerLesErreurs(['nomR' => "Le paramètre 'search' est obligatoire."]);
}

if (!preg_match("/^[\p{L} '-]+$/u", $search)) {

    ReponseJson::envoyerLesErreurs(['nomR' => 'Le nom contient des caractères invalides.']);
}

ReponseJson::envoyerLesDonnees(Coureur::getByNomPrenom($search));
```

Le contrôleur :

1. vérifie la méthode HTTP ;
2. récupère les données ;
3. valide les données ;
4. retourne les éventuelles erreurs ;
5. retourne les données si tout est correct.

Si une exception technique survient pendant le traitement, elle n'est pas capturée par le contrôleur.

Elle remonte jusqu'à `Erreur`.

# 11. Exemple complet d'un script appelé directement

```php
use ClasseMetier\Competence;
use ClasseMetier\CompetenceProjet;
use ClasseMetier\Projet;
use ClasseTechnique\InterfaceSystem;
use ClasseTechnique\Requete;
use ClasseTechnique\Page;

require $_SERVER['DOCUMENT_ROOT'] . "/../bootstrap/bootstrap.php";

// Contrôle et récupération du paramètre attendu : id
$id = Requete::getInt('id');

// Vérification de l'existence du projet
$ligne = Projet::getById($id);

if (!$ligne) {

    $titre = "Projet inexistant";
    $message = "Le projet $id n'existe pas.";
    $type = "erreur";

    $interface = new InterfaceSystem($titre, $message, $type);

    $interface->afficher();
}

$page = new Page();

$page->setTitre("Mise à jour des compétences d'un projet")
    ->setDonnee("projet", $ligne)
    ->setDonnee("lesCompetencesBloc1", Competence::getLesCompetences(1))
    ->setDonnee("lesIdCompetencesProjet", CompetenceProjet::getLesIdCompetences($id))
    ->avecJeton()
    ->afficher();
```

Le script connaît son contexte : il est appelé directement par le navigateur.

Il utilise donc directement :

```php
InterfaceSystem
```

# 12. Le gestionnaire global `Erreur`

La classe `Erreur` constitue le **gestionnaire global des exceptions** de l'application.

Elle est installée avec :

```php
Erreur::installerGestionnaire();
```

Cette méthode installe un gestionnaire d'exceptions PHP :

```php
set_exception_handler(...)
```

Toute exception non interceptée remonte donc vers `Erreur`.

Le traitement suit alors le principe :

```text
Exception
    │
    ▼
  Erreur
    │
    ├── UserException
    │       └── message utilisateur
    │
    ├── PDOException
    │       ├── journalisation
    │       └── traduction du message SQL
    │
    └── autre Throwable
            ├── journalisation
            └── message générique
```

`Erreur` détermine ensuite le type de réponse attendu.

```text
                 Erreur
                    │
             ┌──────┴──────┐
             │             │
            HTML          JSON
             │             │
             ▼             ▼
       page système    ReponseJson
```

# 13. Les pages système HTML

Pour les réponses HTML, l'application utilise :

```php
InterfaceSystem
```

Cette classe est spécialement conçue pour les **pages système**.

Elle est indépendante de `Page` et de `InterfaceHtml`.

Elle permet notamment de générer :

- une page d'erreur ;
- une page d'accès interdit ;
- une page 404 ;
- une page 410 ;
- une page de maintenance ;
- toute autre page système.

## Pourquoi `InterfaceSystem` est indépendante ?

Une page système doit rester minimale.

Elle ne doit pas dépendre de tout le fonctionnement normal de l'application.

`InterfaceHtml` est destinée aux pages normales et peut notamment dépendre de :

```text
Page
templates
JavaScript
styles
menus
données JavaScript
token CSRF
etc.
```

Ces mécanismes sont inutiles pour une page système.

Il faut donc utiliser :

```php
InterfaceSystem
```

et non :

```php
InterfaceHtml
```

---

# 14. La page `/erreur` et les pages HTTP

## La page `/erreur`

Lorsqu'une `UserException` ou une erreur technique doit être présentée en HTML, `Erreur` peut enregistrer temporairement le message en session :

```php
$_SESSION['erreur'] = $message;
```

puis rediriger vers :

```text
/erreur
```

Le fonctionnement est alors :

```text
Exception
    ↓
Erreur
    ↓
$_SESSION['erreur']
    ↓
redirection vers /erreur
    ↓
erreur/index.php
    ↓
récupération du message
    ↓
InterfaceSystem
    ↓
page HTML
```

La page `/erreur` récupère ensuite le message et peut le supprimer :

```php
$message = $_SESSION['erreur'];

unset($_SESSION['erreur']);
```

La redirection étant une nouvelle requête HTTP, la session permet de conserver temporairement le message.
### Important

`/erreur` ne doit pas charger `bootstrap.php`.

Elle doit rester indépendante du gestionnaire global afin d'éviter une récursion du système d'erreurs.

Elle peut charger directement :

```php
require $_SERVER['DOCUMENT_ROOT'] . '/../vendor/autoload.php';
```

puis utiliser :

```php
InterfaceSystem
```

## Pages HTTP spécifiques

Certaines situations ne nécessitent pas de passer par le cycle `/erreur`.

C'est notamment le cas des pages :

```text
403.php
404.php
410.php
```

Ces pages peuvent utiliser directement :

```php
InterfaceSystem
```

Elles sont donc indépendantes du cycle normal de gestion des exceptions.

Exemple :

```php
require $_SERVER['DOCUMENT_ROOT'] . '/../vendor/autoload.php';

$interface = new InterfaceSystem(
    'La page demandée n\'existe pas.',
    'La ressource demandée est introuvable.',
    'erreur'
);

$interface->afficher();
```

Les pages système ne doivent donc pas charger :

```php
bootstrap.php
```

# 15. Transactions SQL et `try / catch`

Un `try / catch` n'est pas nécessaire simplement pour transmettre une exception à `Erreur`.

Par exemple, ceci est inutile :

```php
try {

    $membre = Membre::get($id);

} catch (Throwable $e) {

    throw $e;
}
```

L'exception remontera naturellement vers le gestionnaire global.

En revanche, un `try / catch` est nécessaire lorsqu'une action de compensation doit être réalisée.

Le cas classique est une transaction SQL :

```php
$db = Database::getInstance();

$db->beginTransaction();

try {

    // opérations SQL

    $db->commit();

} catch (Throwable $e) {

    $db->rollBack();

    throw $e;
}
```

Le `catch` est ici justifié car il permet d'effectuer :

```php
$db->rollBack();
```

L'exception doit ensuite être relancée :

```php
throw $e;
```

Elle pourra alors être traitée normalement par `Erreur`.

## Ne jamais retourner le message d'une exception technique

Il ne faut jamais faire :

```php
catch (Throwable $e) {
    return $e->getMessage();
}
```

Cela risque d'exposer :

- le nom d'une table ;
- une requête SQL ;
- un chemin serveur ;
- une information de configuration ;
- une information sur l'architecture interne.

Il faut laisser l'exception remonter :

```php
catch (Throwable $e) {
    throw $e;
}
```

ou effectuer d'abord le traitement de compensation :

```php
catch (Throwable $e) {

    $db->rollBack();

    throw $e;
}
```

# 16. Tableau de décision et principes à retenir

## Tableau de décision

| Question                                                     | Solution                                   |
| ------------------------------------------------------------ | ------------------------------------------ |
| Le script affiche une page HTML directement ?                | `InterfaceSystem`                          |
| Le script est exclusivement AJAX ?                           | `ReponseJson`                              |
| Je retourne des données AJAX ?                               | `envoyerLesDonnees()`                      |
| Je retourne un message de succès AJAX ?                      | `envoyerMessage()`                         |
| Je retourne une erreur globale AJAX ?                        | `envoyerErreur()`                          |
| Je retourne des erreurs de validation ?                      | `envoyerLesErreurs()`                      |
| Je dois construire une réponse JSON particulière ?           | `envoyer()`                                |
| Le même traitement peut être appelé en AJAX et directement ? | `UserException`                            |
| Une erreur technique imprévue survient ?                     | Laisser l'exception remonter               |
| Je dois annuler une transaction après une erreur ?           | `try / catch` puis `rollBack()` et `throw` |
| Je dois créer une page 403, 404 ou 410 ?                     | `InterfaceSystem`                          |
| Je dois afficher une erreur applicative HTML ?               | `/erreur` + `InterfaceSystem`              |

## Principe architectural

Le principe fondamental est la **séparation entre le traitement de l'erreur et sa restitution**.

Le code métier doit simplement signaler le problème lorsqu'il s'agit d'une anomalie destinée à l'utilisateur :

```php
throw new UserException("Le classement demandé n'a pas été trouvé.", 404);
```

Il ne doit pas savoir si la réponse sera :

- une page HTML ;
- une réponse JSON ;
- une page système.

C'est `Erreur` qui adapte la restitution lorsque le contexte d'appel est variable.

```text
                         Traitement PHP
                              │
                              ▼
                    Comment est-il appelé ?
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
        Navigateur          AJAX        Plusieurs contextes
              │               │               │
              ▼               ▼               ▼
     InterfaceSystem      ReponseJson    UserException
              │               │               │
              ▼               ▼               ▼
          HTML             JSON        Erreur adapte
                                      la restitution
```

Pour une erreur technique imprévue :

```text
Erreur technique
       │
       ▼
Laisser remonter
       │
       ▼
     Erreur
       │
       ├── Journalisation
       ├── Masquage des détails
       └── Restitution adaptée
```

## Règles d'or

1. **Ne jamais afficher directement le message d'une exception technique.**
2. **Laisser `Erreur` gérer les exceptions techniques non interceptées.**
3. **Utiliser `UserException` lorsqu'une anomalie destinée à l'utilisateur doit remonter au gestionnaire global.**
4. **Utiliser `ReponseJson` lorsqu'un contrôleur AJAX construit volontairement sa réponse.**
5. **Ne pas utiliser `UserException` systématiquement.**
6. **Utiliser `InterfaceSystem` pour les pages système HTML.**
7. **Ne pas utiliser `InterfaceHtml` pour les pages système.**
8. **Ne pas charger `bootstrap.php` dans une page système indépendante.**
9. **Les `UserException` ne sont pas journalisées par `Erreur`.**
10. **Les `PDOException` sont journalisées.**
11. **Les autres `Throwable` sont journalisés lorsqu'ils ne sont pas des `UserException`.**
12. **Les détails techniques restent dans le journal et ne sont jamais exposés à l'utilisateur.**
13. **Un `try / catch` n'est nécessaire que lorsqu'une action doit être effectuée dans le `catch`, par exemple un `rollBack()`.**
14. **Après un `rollBack()`, toujours relancer l'exception avec `throw`.**
15. **Ne jamais faire `return $e->getMessage()` pour une erreur technique.**
16. **Utiliser un code HTTP cohérent avec la situation lorsqu'une `UserException` est utilisée.**
## Règle finale à retenir

> **Le choix du mécanisme dépend d'abord du contexte de restitution.**
> 
> **Page HTML connue → `InterfaceSystem`**  
> **Réponse AJAX connue → `ReponseJson`**  
> **Contexte d'appel variable → `UserException`**  
> **Problème technique imprévu → laisser l'exception remonter**
> 
> **`UserException` n'est pas une erreur technique : son message est volontairement destiné à l'utilisateur et, avec l'implémentation actuelle de `Erreur`, il n'est pas journalisé.**
> 
> **Les erreurs techniques (`PDOException` et autres `Throwable`) sont journalisées avec leurs détails techniques, puis masquées à l'utilisateur.**