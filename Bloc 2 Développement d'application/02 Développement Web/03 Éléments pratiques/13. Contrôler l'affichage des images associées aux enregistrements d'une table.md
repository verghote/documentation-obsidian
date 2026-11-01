
Lorsqu'une table de données possède une colonne contenant le nom d'une image, il faut distinguer deux problèmes :

1. vérifier que le fichier correspondant existe réellement sur le serveur ;
2. afficher l'image dans le navigateur.

Ces deux opérations peuvent être réalisées côté serveur, côté client, ou en combinant les deux.

Cette documentation présente trois approches possibles à partir du cas des annonces.

# 1. Structure utilisée

La table `annonce` contient notamment une colonne `affiche` qui contient le nom du fichier image associé à l'annonce.

```sql
CREATE TABLE annonce
(
    id          SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    nom         VARCHAR(100)      NOT NULL,
    description TEXT              NOT NULL,
    date        DATE              NOT NULL,
    url         VARCHAR(255)      NULL,
    affiche     VARCHAR(30)       NULL
);
```

Par exemple :

| id  | nom                  | date       | affiche         |
| --- | -------------------- | ---------- | --------------- |
| 1   | Course de printemps  | 2026-04-15 | printemps.jpg   |
| 2   | Championnat régional | 2026-05-10 | championnat.jpg |
| 3   | Meeting d'été        | 2026-07-20 | NULL            |

Les fichiers sont physiquement stockés dans :

```text
public/
└── data/
    └── annonce/
        ├── printemps.jpg
        └── championnat.jpg
```

La colonne `affiche` ne contient donc que le nom du fichier :

```text
printemps.jpg
```

et non son chemin complet.

# 2. Solution 1 : vérifier l'existence du fichier côté serveur

## Principe

La première solution consiste à demander au serveur de vérifier que le fichier indiqué dans la base existe réellement.

La méthode `getLesAnnoncesActives()` effectue cette vérification après avoir récupéré les données de la base.

```php
public static function getLesAnnoncesActives(): array
{
    $sql = <<<SQL
      Select nom, description, date, date_format(date, '%d/%m/%Y') as dateFr, affiche, url
      from annonce 
      where date >= curdate()
      order by date;
    SQL;

    $select = new Select();
    $lesLignes = $select->getRows($sql);

    // Ajout d'une colonne permettant de vérifier l'existence réelle de l'affiche
    foreach ($lesLignes as &$ligne) {
        $ligne['present'] =
            isset($ligne['affiche'])
            && $ligne['affiche'] !== ''
            && is_file(self::DOSSIER_ANNONCE . $ligne['affiche']);
    }

    return $lesLignes;
}
```

Une propriété supplémentaire est ainsi ajoutée à chaque enregistrement :

```json
{
    "nom": "Course de printemps",
    "affiche": "printemps.jpg",
    "present": true
}
```

ou :

```json
{
    "nom": "Course de printemps",
    "affiche": "printemps.jpg",
    "present": false
}
```

## Utilisation côté client

La fonction `creerCarte()` peut alors tester les deux informations :

```javascript
// --- Image ---
divImg.innerHTML = '';

if (element.affiche && element.present) {
    const img = document.createElement('img');

    img.src = "/data/annonce/" + element.affiche;
    img.alt = "Affiche";
    img.style.maxWidth = "100%";

    divImg.appendChild(img);
}
```

L'image n'est donc créée que si :

- un nom de fichier est présent dans la base ;
- le fichier existe réellement sur le serveur.

## Avantages

Cette solution permet au serveur de détecter immédiatement une incohérence entre la base de données et les fichiers présents sur le disque.

Par exemple :

```text
Base de données
        │
        └── affiche = "course.jpg"

Système de fichiers
        │
        └── course.jpg absent
```

Le serveur sait que l'annonce référence un fichier inexistant.

Cela peut être intéressant si l'absence du fichier doit avoir une signification métier ou si l'on souhaite détecter les erreurs de configuration.

## Inconvénients

Cette solution ajoute cependant un traitement qui n'est pas nécessaire pour simplement afficher une image.

Pour chaque annonce, le serveur effectue :

```php
is_file(...)
```

Il faut donc parcourir les fichiers du système pour déterminer si chaque affiche existe.

De plus, la propriété `present` ne signifie pas réellement :

> « Le navigateur pourra afficher cette image. »

Elle signifie uniquement :

> « Le fichier existe à cet emplacement sur le serveur. »

L'accès HTTP pourrait malgré tout être impossible.

La solution mélange donc deux responsabilités :

- récupération des données métier ;
- vérification technique du système de fichiers.

Pour une simple image publique, cette vérification est généralement **plus coûteuse et plus complexe que nécessaire**.

# 3. Solution 2 : laisser le navigateur vérifier l'existence de l'image

Une autre approche consiste à ne plus effectuer de vérification côté serveur.

Le serveur transmet simplement la valeur de `affiche`.

Par exemple :

```json
{
    "nom": "Course de printemps",
    "affiche": "printemps.jpg"
}
```

Le navigateur tente ensuite directement de charger l'image.

La fonction `creerCarte()` peut être écrite ainsi :

```javascript
// --- Image ---
divImg.innerHTML = '';

if (element.affiche) {
    const img = document.createElement('img');

    img.src = "/data/annonce/" + element.affiche;
    img.alt = "Affiche";
    img.style.maxWidth = "100%";

    img.onerror = () => {
        img.remove();
    };

    divImg.appendChild(img);
}
```

## Fonctionnement

Si le fichier existe :

```text
<img>
     ↓
/data/annonce/printemps.jpg
     ↓
image affichée
```

S'il n'existe plus :

```text
<img>
     ↓
/data/annonce/printemps.jpg
     ↓
HTTP 404
     ↓
onerror()
     ↓
suppression de l'image
```

Le serveur n'a donc pas besoin de vérifier préalablement l'existence du fichier.

## Avantages

Cette solution est particulièrement adaptée aux images publiques.

Elle présente plusieurs avantages :

- aucune vérification supplémentaire dans la classe métier ;
- aucune propriété `present` à transmettre au JavaScript ;
- moins de logique côté serveur ;
- utilisation directe du mécanisme HTTP prévu pour les ressources statiques ;
- Apache peut servir directement les images sans passer par PHP ;
- le code est plus simple.

La base reste ainsi centrée sur les données métier :

```text
affiche = "printemps.jpg"
```

et le navigateur se charge de déterminer si la ressource est réellement disponible.

## Limite

Le navigateur effectue malgré tout une requête HTTP pour chaque image.

Cependant, cette requête est nécessaire dans tous les cas : pour afficher une image, le navigateur doit bien récupérer le fichier.

La différence est donc importante :

### Solution 1

```text
Navigateur
    │
    │ demande la page
    ▼
PHP
    │
    ├── requête SQL
    ├── is_file()
    └── génération de la page
    │
    ▼
Navigateur
    │
    │ demande l'image
    ▼
Apache
```

### Solution 2

```text
Navigateur
    │
    │ demande la page
    ▼
PHP
    │
    └── requête SQL
    │
    ▼
Navigateur
    │
    │ demande l'image
    ▼
Apache
```

La solution 2 évite donc une opération serveur préalable qui n'est pas indispensable.

# 4. Solution 3 : protéger le répertoire des images

Une troisième solution consiste à ne pas rendre directement accessible le répertoire :

```text
public/data/annonce/
```

Le navigateur ne pourrait alors plus utiliser directement :

```javascript
img.src = "/data/annonce/" + element.affiche;
```

Il faudrait passer par un script PHP chargé de délivrer l'image.

Par exemple :

```text
annonce/
├── index.php
├── index.js
│
└── ajax/
    └── image.php
```

Le navigateur demanderait :

```text
/annonce/ajax/image.php?id=12
```

Le serveur pourrait alors :

1. rechercher l'annonce ;
2. récupérer le nom du fichier ;
3. vérifier son existence ;
4. vérifier éventuellement les droits d'accès ;
5. envoyer l'image au navigateur.

Le principe serait :

```text
Navigateur
     │
     │ image.php?id=12
     ▼
   PHP
     │
     ├── vérification de l'annonce
     ├── vérification des droits
     ├── recherche du fichier
     │
     ▼
 fichier image
     │
     ▼
 Navigateur
```

## Pourquoi protéger un répertoire ?

Cette solution est utile lorsque les fichiers ne doivent **pas être accessibles librement par leur URL**.

Par exemple :

- documents personnels ;
- fichiers réservés aux utilisateurs connectés ;
- documents soumis à des droits d'accès ;
- fichiers confidentiels ;
- ressources dont l'accès doit être contrôlé par l'application.

Dans ce cas, le contrôle côté serveur est justifié.

##  Cette protection ne permet pas d'empêcher la copie d'une image

Il faut cependant bien comprendre la limite fondamentale de cette solution.

Si le serveur autorise un utilisateur à afficher une image, l'image est nécessairement transmise à son navigateur.

L'utilisateur peut alors :

- enregistrer l'image ;
- utiliser les outils de développement du navigateur ;
- récupérer la requête réseau ;
- conserver une copie de l'image.

Protéger le répertoire ne permet donc pas de protéger une image **contre sa copie par un utilisateur autorisé**.

La protection permet seulement de contrôler :

> **Qui a le droit d'obtenir l'image ?**

Elle ne permet pas de contrôler :

> **Ce que l'utilisateur fait avec l'image après l'avoir obtenue.**


##  La solution avec un contrôleur PHP ajoute également de la charge

Avec un accès direct, Apache peut servir le fichier statique :

```text
/data/annonce/printemps.jpg
```

Avec un contrôleur PHP, chaque demande passe par l'application : img.src = "/annonce/image.php?id=12";

```text
image.php?id=12
       │
       ▼
    PHP
       │
       ▼
  readfile(...)
       │
       ▼
 image
```

Cela ajoute :

- l'exécution de PHP ;
- la recherche de l'enregistrement ;
- la vérification du fichier ;
- éventuellement la vérification des droits ;
- l'envoi du fichier depuis PHP.

Pour quelques images publiques, cette complexité n'apporte donc aucun bénéfice réel.

Elle devient pertinente uniquement lorsqu'un **contrôle d'accès est nécessaire**.

# 7. Quelle solution choisir ?

Le choix dépend principalement de la nature des images.

|Situation|Solution recommandée|
|---|---|
|Images publiques|**Accès direct + `onerror`**|
|Images publiques mais contrôle de cohérence métier nécessaire|Vérification serveur|
|Images privées|Contrôleur PHP|
|Images soumises à des droits d'accès|Contrôleur PHP|
|Documents confidentiels|Répertoire protégé + contrôleur|
|Besoin d'empêcher la copie par l'utilisateur|Impossible si l'image lui est affichée|

Pour les affiches des annonces, les images étant destinées à être publiquement affichées, **la deuxième solution est généralement la plus adaptée**.

# 8. Recommandation pour les annonces

Dans le cas de la table `annonce`, il n'est pas nécessaire d'ajouter la propriété `present` aux données retournées par `getLesAnnoncesActives()`.

La classe métier peut simplement retourner les données :

```php
public static function getLesAnnoncesActives(): array
{
    $sql = <<<SQL
        SELECT nom,
               description,
               date,
               DATE_FORMAT(date, '%d/%m/%Y') AS dateFr,
               affiche,
               url
        FROM annonce
        WHERE date >= CURDATE()
        ORDER BY date;
    SQL;

    return (new Select())->getRows($sql);
}
```

Et le JavaScript peut gérer l'affichage :

```javascript
if (element.affiche) {
    const img = document.createElement('img');

    img.src = "/data/annonce/" + element.affiche;
    img.alt = "Affiche";
    img.style.maxWidth = "100%";

    img.onerror = () => {
        img.remove();
    };

    divImg.appendChild(img);
}
```

Cette solution respecte davantage la séparation des responsabilités :

```text
PHP
 │
 └── fournit les données de l'annonce
          │
          ▼
JavaScript
 │
 └── construit l'interface
          │
          ▼
Navigateur
 │
 └── charge les ressources nécessaires
```

La vérification de l'existence du fichier n'est alors effectuée que lorsque le navigateur tente réellement de charger l'image.

# 9. Conclusion

La présence d'un nom de fichier dans une base de données ne garantit pas que le fichier existe physiquement.

Trois stratégies sont possibles :

1. **Vérifier côté serveur avant d'envoyer les données** : permet de connaître immédiatement les incohérences entre la base et le système de fichiers, mais ajoute un traitement qui n'est pas nécessaire pour une simple image publique.
    
2. **Laisser le navigateur gérer l'absence du fichier avec `onerror`** : solution simple et adaptée aux images publiques. Le serveur fournit uniquement les données et le navigateur tente de charger la ressource.
    
3. **Protéger le répertoire et servir les images par PHP** : utile lorsqu'un contrôle d'accès est nécessaire, mais plus lourd. Cette solution ne protège pas l'image contre sa copie par un utilisateur qui est autorisé à l'afficher.
    

Pour les affiches d'annonces publiques, la solution recommandée est donc :

```text
Base de données
      │
      │ affiche = "course.jpg"
      ▼
JavaScript
      │
      │ img.src
      ▼
Apache
      │
      ▼
/data/annonce/course.jpg
```

avec une gestion de `onerror` si l'on souhaite éviter l'affichage d'une image cassée.

La protection du répertoire doit être réservée aux ressources pour lesquelles **l'accès lui-même doit être contrôlé**. Elle ne constitue pas un mécanisme de protection contre la copie d'une image déjà envoyée au navigateur.