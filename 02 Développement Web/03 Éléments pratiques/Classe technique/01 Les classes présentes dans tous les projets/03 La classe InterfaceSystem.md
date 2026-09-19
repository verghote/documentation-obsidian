## 1. Rôle de la classe

La classe `InterfaceSystem` appartient au namespace :

```php
ClasseTechnique
```

Son rôle est de **générer et afficher une page système autonome**, intégrée visuellement à la charte graphique du site.

Elle est destinée notamment à afficher :

- les erreurs ;
- les avertissements ;
- les informations système ;
- les messages de maintenance.

Sa particularité essentielle est qu'elle est **indépendante du système normal de génération des pages**.

La documentation de la classe précise :

> Cette classe est indépendante de `bootstrap.php` et d'`InterfaceHtml` afin de pouvoir être utilisée notamment lors de la gestion des erreurs.

Cette indépendance est importante : si une erreur empêche le fonctionnement normal de l'application, `InterfaceSystem` doit rester utilisable.

---

# 2. Positionnement dans l'architecture

Le fonctionnement général peut être représenté ainsi :

```text
                    Application
                         │
                         │ fonctionnement normal
                         ▼
                  InterfaceHtml
                         │
                         ▼
                  Page applicative
```

En cas de problème :

```text
                    Erreur
                      │
                      ▼
                Répertoire erreur/
                      │
                      ▼
                InterfaceSystem
                      │
                      ▼
          interface_system.php
                      │
                      ▼
              Page système HTML
```

`InterfaceSystem` constitue donc une **voie d'affichage alternative** au système normal d'interface.

---

# 3. Pourquoi une interface indépendante ?

Le système normal repose notamment sur :

```text
bootstrap.php
InterfaceHtml
Page
configuration
fragments
```

Or une erreur peut précisément empêcher certains de ces éléments de fonctionner correctement.

Il serait donc risqué d'utiliser `InterfaceHtml` pour afficher une erreur qui survient pendant son propre fonctionnement.

`InterfaceSystem` limite volontairement ses dépendances.

Son principe est :

```text
Erreur
  │
  ├── ne dépend pas de Page
  ├── ne dépend pas de InterfaceHtml
  ├── ne dépend pas du bootstrap applicatif
  │
  ▼
InterfaceSystem
  │
  ▼
Template système
```

---

# 4. Structure de la classe

La classe contient trois propriétés principales :

```php
private string $message;
private string $titre;
private string $type;
```

Elles représentent les trois informations nécessaires à l'affichage :

|Propriété|Contenu|
|---|---|
|`$message`|Message à afficher|
|`$titre`|Titre de la page|
|`$type`|Type du message système|

---

# 5. Types de messages

La classe définit une constante :

```php
private const TYPES = [
    'erreur' => 'Erreur',
    'avertissement' => 'Avertissement',
    'information' => 'Information',
    'maintenance' => 'Maintenance',
];
```

Elle définit les types de messages reconnus par l'interface.

Les valeurs acceptées sont donc :

```text
erreur
avertissement
information
maintenance
```

Chaque type possède également un libellé :

```text
erreur        → Erreur
avertissement → Avertissement
information   → Information
maintenance   → Maintenance
```

La constante est `private`, ce qui signifie qu'elle est utilisable uniquement à l'intérieur de la classe.

---

# 6. Constructeur

Le constructeur est :

```php
public function __construct(
    string $titre,
    string $message,
    string $type = 'avertissement'
)
```

Il reçoit trois paramètres :

```text
titre
message
type
```

Le troisième paramètre est facultatif.

S'il n'est pas fourni, le type utilisé est :

```text
avertissement
```

---

# 7. Stockage du message et du titre

Le constructeur enregistre directement les informations reçues :

```php
$this->message = $message;
$this->titre = $titre;
```

Elles seront utilisées plus tard lors de l'affichage de la page.

---

# 8. Validation du type

Le type fourni au constructeur est contrôlé :

```php
$this->type =
    array_key_exists($type, self::TYPES)
        ? $type
        : 'avertissement';
```

Cela signifie que seuls les types définis dans `TYPES` sont acceptés.

Exemple valide :

```php
new InterfaceSystem(
    'Erreur',
    'Une erreur est survenue.',
    'erreur'
);
```

Le type est :

```text
erreur
```

---

# 9. Type inconnu

Si un type qui n'existe pas est fourni :

```php
new InterfaceSystem(
    'Message',
    'Contenu',
    'autre'
);
```

le type est automatiquement remplacé par :

```text
avertissement
```

Ainsi, la classe garantit que `$this->type` possède toujours une valeur reconnue.

---

# 10. Pourquoi valider le type ?

La validation évite de transmettre au template une valeur arbitraire.

Le template peut donc supposer que `$type` contient toujours l'une des valeurs suivantes :

```text
erreur
avertissement
information
maintenance
```

Cela simplifie le traitement côté HTML/CSS.

Par exemple, le template peut utiliser le type pour choisir une présentation adaptée :

```text
erreur        → présentation rouge
avertissement → présentation orange
information   → présentation bleue
maintenance   → présentation spécifique
```

La manière exacte dont le type est utilisé visuellement dépend toutefois du fichier :

```text
view/interface_system.php
```

---

# 11. Méthode `afficher()`

La méthode publique :

```php
public function afficher(): void
```

est responsable de l'affichage de la page système.

Elle ne retourne pas de chaîne HTML.

Elle effectue directement l'affichage puis termine l'exécution.

---

# 12. Préparation des variables du template

La méthode commence par recopier les propriétés dans des variables locales :

```php
$message = $this->message;
$titre = $this->titre;
$type = $this->type;
```

Ces variables sont destinées au template.

Le fichier `interface_system.php` pourra donc utiliser :

```php
$message
$titre
$type
```

---

# 13. Détermination du template

Le fichier utilisé pour l'affichage est déterminé avec :

```php
$template =
    dirname(__DIR__, 2) .
    '/view/interface_system.php';
```

La classe recherche donc le template :

```text
view/interface_system.php
```

à partir de la structure du projet.

Le template est indépendant des templates utilisés par `InterfaceHtml`.

---

# 14. Chargement du template

Le template est exécuté avec :

```php
require $template;
```

Les variables préparées juste avant :

```php
$message
$titre
$type
```

sont disponibles dans le fichier inclus.

Le fonctionnement est donc :

```text
InterfaceSystem
      │
      ├── $titre
      ├── $message
      └── $type
             │
             ▼
view/interface_system.php
             │
             ▼
       HTML généré
```

---

# 15. Fin immédiate de l'exécution

Après le chargement du template :

```php
exit;
```

met immédiatement fin à l'exécution du script PHP.

C'est un choix important pour une page système.

Une fois la page d'erreur affichée, l'application ne doit pas continuer à exécuter le traitement qui a conduit à l'erreur.

Le cycle est donc :

```text
Erreur détectée
      │
      ▼
Création InterfaceSystem
      │
      ▼
afficher()
      │
      ▼
Chargement du template
      │
      ▼
Affichage
      │
      ▼
exit
      │
      X
Fin du traitement
```

---

# 16. Exemple minimal d'utilisation

La classe peut être utilisée ainsi :

```php
use ClasseTechnique\InterfaceSystem;

$interface = new InterfaceSystem(
    'Erreur',
    'Une erreur est survenue.',
    'erreur'
);

$interface->afficher();
```

Le constructeur prépare les données et `afficher()` génère la page système.

---

# 17. Utilisation par `erreur/index.php`

Le fichier fourni :

```text
erreur/index.php
```

constitue un point d'affichage des erreurs.

Il commence par charger Composer :

```php
require $_SERVER['DOCUMENT_ROOT'] . '/../vendor/autoload.php';
```

Cela permet notamment de rendre disponibles les classes utilisées :

```php
use ClasseTechnique\InterfaceSystem;
use ClasseTechnique\Journal;
```

---

# 18. Démarrage de la session dans `erreur/index.php`

Le fichier démarre également la session si nécessaire :

```php
if (session_status() === PHP_SESSION_NONE) {
    session_start();
}
```

Cette opération est nécessaire car le message d'erreur est récupéré depuis :

```php
$_SESSION['erreur']
```

---

# 19. Définition des valeurs par défaut

Le fichier définit :

```php
$titre = 'Erreur';
$type = 'erreur';
```

Dans ce contexte, la page est donc destinée à afficher une erreur.

Le commentaire indique que les erreurs provenant d'une exception utilisent ces valeurs par défaut.

Le message proprement dit est récupéré séparément.

---

# 20. Récupération du message d'erreur

Le fichier vérifie :

```php
if (
    isset($_SESSION['erreur'])
    && is_string($_SESSION['erreur'])
)
```

Deux conditions doivent donc être réunies :

1. la variable de session existe ;
2. sa valeur est une chaîne de caractères.

Si c'est le cas :

```php
$message = $_SESSION['erreur'];
```

Le message est récupéré.

---

# 21. Suppression du message de session

Après récupération :

```php
unset($_SESSION['erreur']);
```

La variable de session est supprimée.

Cela empêche que le même message soit réutilisé lors d'un affichage ultérieur.

Le fonctionnement est donc :

```text
$_SESSION['erreur']
       │
       ▼
 récupération
       │
       ▼
$message
       │
       ▼
unset($_SESSION['erreur'])
```

Le message est ainsi traité comme une information temporaire.

---

# 22. Journalisation de l'erreur

Le message est ensuite transmis à :

```php
Journal::enregistrer(
    $message,
    'erreur'
);
```

La classe `Journal` est donc chargée d'enregistrer l'erreur.

`InterfaceSystem` n'a pas cette responsabilité.

La séparation est claire :

```text
Journal
   │
   └── enregistre l'erreur

InterfaceSystem
   │
   └── affiche l'erreur
```

---

# 23. Cas où aucun message n'est disponible

Si :

```php
$_SESSION['erreur']
```

n'existe pas ou n'est pas une chaîne, le fichier utilise :

```php
$message =
    'Une erreur inconnue est survenue.';
```

La page peut donc toujours afficher un message, même si le message d'origine n'est plus disponible.

Le fonctionnement est :

```text
                $_SESSION['erreur']
                       │
              ┌────────┴────────┐
              │                 │
          disponible         absent/invalide
              │                 │
              ▼                 ▼
        message réel       message générique
              │                 │
              └────────┬────────┘
                       ▼
                  InterfaceSystem
```

---

# 24. Création de `InterfaceSystem`

Une fois le message déterminé :

```php
$interface =
    new InterfaceSystem(
        $titre,
        $message,
        $type
    );
```

L'objet possède alors :

```text
titre   = Erreur
message = message récupéré
type    = erreur
```

---

# 25. Affichage de la page

L'affichage est déclenché par :

```php
$interface->afficher();
```

Cette méthode :

1. prépare les variables ;
2. détermine le template ;
3. charge `view/interface_system.php` ;
4. affiche la page ;
5. arrête l'exécution avec `exit`.

---

# 26. Architecture complète de la gestion d'erreur

Le fonctionnement peut être résumé ainsi :

```text
Erreur / Exception
       │
       ▼
Enregistrement temporaire
dans $_SESSION['erreur']
       │
       ▼
erreur/index.php
       │
       ├── récupère le message
       ├── supprime $_SESSION['erreur']
       └── Journal::enregistrer()
                │
                ▼
       InterfaceSystem
                │
                ├── titre
                ├── message
                └── type
                │
                ▼
view/interface_system.php
                │
                ▼
          Page système
                │
                ▼
              exit
```

---

# 27. Différence avec `InterfaceHtml`

Les deux classes ont des responsabilités différentes.

|`InterfaceHtml`|`InterfaceSystem`|
|---|---|
|Interface normale de l'application|Interface système|
|Utilise `Page`|N'utilise pas `Page`|
|Construit le `<head>` dynamiquement|Utilise un template système|
|Gère CSS/scripts/menus|Affiche principalement un message|
|Utilisée par les pages normales|Utilisée notamment pour les erreurs|
|Dépend de l'architecture applicative|Fonctionnement volontairement indépendant|
|Retourne des fragments HTML|`afficher()` affiche et termine le script|

Cette séparation est importante pour la robustesse du système.

---

# 28. Rôle du fichier `view/interface_system.php`

`InterfaceSystem` ne contient pas directement le HTML de la page système.

Elle délègue cette responsabilité à :

```text
view/interface_system.php
```

La classe prépare simplement :

```php
$titre
$message
$type
```

Le template est responsable de leur présentation.

On retrouve donc une séparation :

```text
InterfaceSystem
       │
       │ données
       ▼
interface_system.php
       │
       │ présentation
       ▼
HTML
```

---

# 29. Séparation des responsabilités

L'architecture distingue plusieurs responsabilités :

```text
erreur/index.php
    │
    ├── récupération du message
    ├── gestion de $_SESSION
    └── journalisation
             │
             ▼
InterfaceSystem
    │
    ├── validation du type
    ├── préparation des données
    └── chargement du template
             │
             ▼
interface_system.php
    │
    └── présentation HTML
```

Cette séparation évite de mélanger :

- récupération de l'erreur ;
- journalisation ;
- logique d'affichage ;
- présentation HTML.

---

# 30. Points importants à retenir

## `InterfaceSystem`

Son rôle est de fournir une **interface minimale et autonome pour les pages système**.

Elle reçoit :

```text
titre
message
type
```

valide le type, puis charge :

```text
view/interface_system.php
```

Enfin, elle arrête l'exécution avec :

```php
exit;
```

## `erreur/index.php`

Son rôle est de :

1. charger les classes ;
2. démarrer la session ;
3. récupérer le message d'erreur ;
4. supprimer le message de la session ;
5. enregistrer l'erreur dans le journal ;
6. créer `InterfaceSystem` ;
7. afficher la page système.

---

# 31. Résumé

|Élément|Responsabilité|
|---|---|
|`erreur/index.php`|Point d'entrée pour l'affichage d'une erreur|
|`$_SESSION['erreur']`|Transport temporaire du message|
|`Journal`|Enregistrement de l'erreur|
|`InterfaceSystem`|Préparation et affichage de la page système|
|`TYPES`|Liste des types de messages autorisés|
|`interface_system.php`|Présentation HTML de la page système|
|`exit`|Arrêt de l'exécution après affichage|

---

# 32. Principe architectural

Le principe fondamental est :

> **L'interface système ne doit pas dépendre du système d'interface normal qu'elle peut être amenée à remplacer en cas d'erreur.**

On dispose ainsi de deux circuits distincts :

```text
                 APPLICATION
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
   Fonctionnement normal      Erreur
          │                     │
          ▼                     ▼
   InterfaceHtml          InterfaceSystem
          │                     │
          ▼                     ▼
    Page normale          Page système
```

Cette architecture permet à l'application de conserver un moyen d'afficher un message compréhensible même lorsque le circuit normal de génération des pages n'est plus utilisable.