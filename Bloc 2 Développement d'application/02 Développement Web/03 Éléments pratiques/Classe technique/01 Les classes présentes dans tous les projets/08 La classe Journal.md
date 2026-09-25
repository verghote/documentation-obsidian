## Présentation

La classe `Journal` centralise la journalisation des événements de l'application.

Elle permet d'enregistrer automatiquement des informations dans des fichiers de journal (_logs_) afin de conserver une trace des actions importantes de l'application.

Les journaux sont créés automatiquement dans un répertoire dédié et ne nécessitent aucun fichier de configuration.

La classe est entièrement statique : aucune instance n'est créée.

---
# Rôle dans l'architecture

La journalisation est utilisée par plusieurs composants du framework.

```text
                 Application
                      │
      ┌───────────────┼───────────────┐
      │               │               │
      ▼               ▼               ▼
   Erreur        Sécurité        Métier
      │               │               │
      └───────────────┴───────────────┘
                      │
                      ▼
                  Journal
                      │
                      ▼
              Fichiers *.log
```

La classe permet notamment de conserver la trace :

- des erreurs techniques ;
    
- des tentatives d'intrusion ;
    
- des actions utilisateur ;
    
- des événements métier ;
    
- des traitements planifiés.
    

---

# Une classe utilitaire

La classe est exclusivement composée de méthodes statiques.

Aucun objet n'est créé.

Toutes les opérations s'effectuent directement par :

```php
Journal::enregistrer(...);
```

ou

```php
Journal::ecrire(...);
```

---
# Organisation des journaux

Tous les journaux sont stockés dans le répertoire :

```text
../log
```

par rapport au dossier `public` (`DOCUMENT_ROOT`).

Exemple :

```text
Projet
│
├── public
│
└── log
    ├── erreur.log
    ├── menace.log
    ├── evenement.log
    └── connexion.log
```

Le dossier est créé automatiquement lors du premier enregistrement.

Aucune configuration préalable n'est nécessaire.

---

# Création automatique des journaux

Le chemin d'un journal est obtenu grâce à la méthode privée :

```php
getChemin()
```

Cette méthode :

- construit le chemin absolu du fichier ;
    
- crée le dossier `log` s'il n'existe pas ;
    
- retourne le chemin complet du fichier.
    

Ainsi, il suffit d'écrire :

```php
Journal::enregistrer(..., 'connexion');
```

pour que le fichier

```text
connexion.log
```

soit créé automatiquement.

---

# Écriture brute

La méthode

```php
ecrire()
```

écrit un texte sans aucune mise en forme.

Prototype :

```php
Journal::ecrire(
    string $message,
    string $journal = 'evenement'
);
```

Exemple :

```php
Journal::ecrire(
    "Début du traitement\n",
    "import"
);
```

Le texte est enregistré tel quel.

Cette méthode est utile lorsqu'un format particulier est souhaité ou lorsqu'un contenu volumineux doit être enregistré.

---

# Écriture formatée

La méthode

```php
enregistrer()
```

est celle utilisée dans la majorité des cas.

Elle ajoute automatiquement des informations de contexte.

Prototype :

```php
Journal::enregistrer(
    string $evenement,
    string $journal = 'evenement'
);
```

Chaque ligne est automatiquement enrichie avec :

- la date ;
    
- le script ayant généré l'événement ;
    
- l'adresse IP du client.
    

---

# Format des journaux

Chaque événement est enregistré sous la forme :

```text
date    événement    script    adresse IP
```

Exemple :

```text
03/07/2026 14:32:08    Connexion réussie    /connexion/index.php    192.168.1.12
```

Les champs sont séparés par des tabulations.

Ce format est :

- facilement lisible ;
    
- facilement exploitable dans Excel ;
    
- facilement importable dans un autre outil.
    

---

# Formatage des lignes

Le format est généré par la méthode privée :

```php
formatterLigne()
```

Elle construit une ligne contenant :

- la date et l'heure ;
    
- le texte de l'événement ;
    
- le script appelant ;
    
- l'adresse IP.
    

L'ensemble est ensuite transmis à la méthode d'écriture.

---

# Verrouillage des fichiers

Avant chaque écriture, la classe verrouille le fichier grâce à :

```php
flock(..., LOCK_EX)
```

Cette précaution évite les conflits lorsque plusieurs requêtes tentent d'écrire simultanément dans le même journal.

Le verrou est libéré automatiquement après l'écriture.

Cette approche garantit l'intégrité des fichiers de journal.

---

# Lecture d'un journal

La méthode

```php
getLesEvenements()
```

retourne l'ensemble des lignes d'un journal.

Prototype :

```php
Journal::getLesEvenements(
    string $journal = 'evenement'
);
```

Si le journal n'existe pas :

```php
[]
```

est retourné.

Le résultat est un tableau contenant une ligne par événement.

Exemple :

```php
[
    "03/07/2026 ...",
    "03/07/2026 ...",
    "02/07/2026 ..."
]
```

Cette méthode est utilisée pour afficher le contenu d'un journal dans une interface d'administration.

---

# Suppression d'un journal

La méthode

```php
supprimer()
```

supprime complètement un fichier de journal.

Exemple :

```php
Journal::supprimer('erreur');
```

Le fichier

```text
erreur.log
```

est alors supprimé.

Si le fichier n'existe pas, aucune erreur n'est générée.

---

# Liste des journaux

La méthode

```php
getListe()
```

recherche automatiquement tous les fichiers `*.log`.

Elle retourne un tableau associatif.

Exemple :

```php
[
    'erreur'    => 'Journal erreur',
    'menace'    => 'Journal menace',
    'connexion' => 'Journal connexion'
]
```

Cette méthode permet de construire automatiquement une liste déroulante ou un menu des journaux disponibles.

Aucun fichier n'a besoin d'être déclaré manuellement.

---

# Détermination de l'adresse IP

La méthode

```php
getIp()
```

détermine la meilleure adresse IP disponible.

Les en-têtes suivants sont examinés dans l'ordre :

1. `HTTP_X_FORWARDED_FOR`
    
2. `HTTP_CLIENT_IP`
    
3. `REMOTE_ADDR`
    

Si plusieurs adresses sont présentes dans `HTTP_X_FORWARDED_FOR`, seule la première est conservée.

Toutes les adresses sont validées grâce à :

```php
filter_var()
```

Les adresses publiques sont privilégiées.

Les adresses privées sont acceptées lorsqu'aucune adresse publique n'est disponible.

En mode ligne de commande, la méthode retourne :

```text
CLI
```

---

# Cycle d'un enregistrement

```text
Journal::enregistrer()
          │
          ▼
    getChemin()
          │
          ▼
 formatterLigne()
          │
          ▼
 ouverture du fichier
          │
          ▼
 verrouillage (LOCK_EX)
          │
          ▼
    écriture
          │
          ▼
déverrouillage
          │
          ▼
 fermeture
```

---

# Exemples d'utilisation

## Journal d'erreurs

```php
Journal::enregistrer(
    $message,
    'erreur'
);
```

---

## Journal des tentatives d'intrusion

```php
Journal::enregistrer(
    $_SERVER['REQUEST_URI'],
    'menace'
);
```

---

## Journal métier

```php
Journal::enregistrer(
    "Création d'un nouveau licencié",
    'membre'
);
```

---

## Journal d'import

```php
Journal::ecrire(
    "Import terminé\n",
    'import'
);
```

---

# Intégration dans le framework

La classe est utilisée notamment par :

|Classe|Utilisation|
|---|---|
|`Erreur`|Journalisation des exceptions techniques|
|`Erreur::bloquerVisiteur()`|Journalisation des tentatives malveillantes|
|Classes métier|Journalisation d'événements fonctionnels|
|Scripts d'administration|Journalisation des traitements automatiques|

Elle constitue ainsi le composant unique de journalisation du framework.

---

# Avantages de cette architecture

- Classe entièrement statique, simple à utiliser.
    
- Création automatique des dossiers et des journaux.
    
- Aucun fichier de configuration nécessaire.
    
- Journalisation homogène dans toute l'application.
    
- Format de journal lisible et facilement exploitable.
    
- Verrouillage des fichiers garantissant l'intégrité des écritures concurrentes.
    
- Détection automatique de l'adresse IP du client.
    
- Découverte automatique des journaux existants.
    
- Réutilisable aussi bien pour les événements techniques que pour les événements métier.