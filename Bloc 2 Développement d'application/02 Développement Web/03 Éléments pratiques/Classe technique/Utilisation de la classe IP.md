## 1. Présentation

La classe `Ip` fournit les fonctionnalités nécessaires à la **gestion des adresses IP bloquées** par l'application.

Elle permet notamment :

- d'identifier l'adresse IP du client ;
- de vérifier si une adresse IP fait actuellement l'objet d'un blocage ;
- de récupérer les informations associées à un blocage ;
- de bloquer une adresse IP pour une durée déterminée ;
- de consulter l'historique des blocages.

La classe est conçue pour être utilisée par les différents mécanismes de sécurité de l'application.

Une IP peut être bloquée pour différentes raisons, par exemple :

- nombre excessif de requêtes ;
- tentative de traversée de répertoire ;
- tentative d'injection SQL ;
- tentative d'injection de code ;
- comportement automatisé suspect ;
- toute autre anomalie nécessitant un blocage temporaire.

La classe `Ip` **ne détermine pas pourquoi une IP doit être bloquée**. Elle fournit uniquement le mécanisme permettant de gérer ce blocage.

Par exemple, la détection d'un trafic excessif est réalisée par la classe `TraficIp` :

```php
if (TraficIp::surveiller($ip)) {
    // l'IP vient d'être bloquée
}
```

La classe `TraficIp` peut alors demander à `Ip` d'effectuer le blocage :

```php
Ip::bloquer($ip, 'Plus de 10 requêtes en moins de 5 secondes',  600);
```

Cette séparation permet à d'autres mécanismes de sécurité d'utiliser exactement le même système :

```php
Ip::bloquer($ip, 'Tentative de traversée de répertoire', 3600);
```

ou :

```php
Ip::bloquer($ip, 'Tentative d\'injection SQL', 3600);
```

---

# 2. Dépendance à la base de données

La classe `Ip` nécessite la présence d'une table `ipbloquee`.

La structure attendue est la suivante :

```sql
CREATE TABLE ipbloquee (
    id          INT UNSIGNED NOT NULL AUTO_INCREMENT,
    ip          VARCHAR(45)  NOT NULL,
    dateblocage DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
    motifblocage VARCHAR(255) NOT NULL,
    dureeblocage INT          NOT NULL,

    PRIMARY KEY (id),

    INDEX idx_ip_dureeblocage (ip, dateblocage)
);
```

## Description des colonnes

| Colonne        | Description                                    |
| -------------- | ---------------------------------------------- |
| `id`           | Identifiant unique du blocage                  |
| `ip`           | Adresse IP concernée                           |
| `dateblocage`  | Date et heure auxquelles le blocage a été créé |
| `motifblocage` | Motif ayant entraîné le blocage                |
| `dureeblocage` | Durée du blocage en secondes                   |

La date de fin n'est volontairement pas stockée dans la table.

Elle est calculée à partir de :

```text
dateblocage + dureeblocage
```

Par exemple :

```text
dateblocage  = 2026-08-28 15:00:00
dureeblocage = 600 secondes

date de fin = 2026-08-28 15:10:00
```

Cela évite de stocker deux informations redondantes.

---

# 3. Index sur l'adresse IP

La table possède l'index :

```sql
INDEX idx_ip_dureeblocage (ip, dateblocage)
```

Cet index est important car les recherches effectuées par la classe portent principalement sur une adresse IP :

```sql
WHERE ip = :ip
```

et sur la date du blocage :

```sql
AND dateblocage + INTERVAL dureeblocage SECOND > NOW()
```

L'index permet donc de retrouver rapidement les enregistrements correspondant à une adresse IP.

Il est important de noter que **l'adresse IP n'est pas unique**.

Une même IP peut être bloquée plusieurs fois au cours du temps, avec éventuellement des motifs et des durées différents.

La table constitue donc également un **historique des blocages**.

---

# 4. Principe général

Le fonctionnement peut être résumé ainsi :

```text
              Détection d'une anomalie
                       │
                       ▼
                  Ip::bloquer()
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
      Journalisation       Enregistrement
        de la menace       dans ipbloquee
                                  │
                                  ▼
                         Blocage temporaire
```

Lors d'une requête suivante, le code de l'application peut vérifier :

```php
$blocage = Ip::getBlocage($ip);
```

Si un blocage actif existe, la méthode retourne ses informations.

Sinon, elle retourne `null`.

---

# 5. `Ip::getIp()`

## Rôle

Retourne l'adresse IP du client.

```php
$ip = Ip::getIp();
```

La méthode examine plusieurs variables HTTP :

```text
HTTP_X_FORWARDED_FOR
HTTP_CLIENT_IP
REMOTE_ADDR
```

Elle tente de récupérer une adresse IP valide.

Pour `HTTP_X_FORWARDED_FOR`, plusieurs adresses peuvent être présentes. La méthode utilise la première adresse de la liste.

Si aucune adresse IP valide n'est trouvée, la méthode retourne :

```text
CLI
```

Cette valeur permet notamment de distinguer une exécution en ligne de commande d'une requête HTTP.

### Exemple

```php
$ip = Ip::getIp();

echo $ip;
```

Résultat possible :

```text
192.168.1.25
```

ou, en exécution CLI :

```text
CLI
```

> **Attention :** les en-têtes comme `X-Forwarded-For` ne doivent être considérés comme fiables que si l'application est placée derrière un proxy ou un reverse proxy maîtrisé et configuré pour les renseigner correctement. Dans le cas contraire, un client peut falsifier ces en-têtes.

---

# 6. `Ip::getBlocage()`

## Rôle

Recherche le blocage **actuellement actif** pour une adresse IP.

```php
$blocage = Ip::getBlocage($ip);
```

La méthode recherche les blocages correspondant à l'adresse IP et dont la date de fin n'est pas dépassée.

La date de fin est calculée ainsi :

```sql
dateblocage + INTERVAL dureeblocage SECOND
```

La méthode retourne :

- un tableau contenant les informations du blocage si l'IP est actuellement bloquée ;
    
- `null` si aucun blocage actif n'existe.
    

### Exemple

```php
$blocage = Ip::getBlocage($ip);

if ($blocage !== null) {
    echo $blocage['motifblocage'];
}
```

Le tableau retourné contient notamment :

```php
[
    'motifblocage'  => 'Tentative de traversée de répertoire',
    'dureeblocage'  => 3600,
    'dateblocage'   => '2026-08-28 16:20:00',
    'datefinblocage' => '2026-08-28 17:20:00'
]
```

---

# 7. Pourquoi `getBlocage()` retourne `null` ?

La signature de la méthode est :

```php
public static function getBlocage(string $ip): ?array
```

Le `?array` signifie que la méthode peut retourner :

```text
array
```

ou :

```text
null
```

Cela permet d'utiliser naturellement le résultat :

```php
$blocage = Ip::getBlocage($ip);

if ($blocage !== null) {
    // L'IP est actuellement bloquée.
}
```

Le développeur n'a donc pas besoin de faire une deuxième requête pour déterminer si l'IP est bloquée.

---

# 8. `Ip::estBloquee()`

## Rôle

Permet de savoir simplement si une IP est actuellement bloquée.

```php
if (Ip::estBloquee($ip)) {
    // IP bloquée
}
```

Cette méthode s'appuie sur `getBlocage()` :

```php
return self::getBlocage($ip) !== null;
```

Elle est donc particulièrement pratique lorsqu'on n'a pas besoin du motif du blocage.

### Exemple

```php
$ip = Ip::getIp();

if (Ip::estBloquee($ip)) {
    throw new UserException(
        'Votre adresse IP est temporairement bloquée.'
    );
}
```

Si le motif doit être affiché, il est préférable d'utiliser directement `getBlocage()` :

```php
$blocage = Ip::getBlocage($ip);

if ($blocage !== null) {
    throw new UserException(
        'Votre adresse IP est temporairement bloquée : '
        . $blocage['motifblocage']
    );
}
```

---

# 9. `Ip::bloquer()`

## Rôle

Bloque une adresse IP pour une durée donnée.

```php
Ip::bloquer($ip, $motif, $duree);
```

La méthode reçoit trois paramètres :

|Paramètre|Type|Description|
|---|---|---|
|`$ip`|`string`|Adresse IP à bloquer|
|`$motif`|`string`|Raison du blocage|
|`$duree`|`int`|Durée du blocage en secondes|

### Exemple

Bloquer une IP pendant 10 minutes :

```php
Ip::bloquer(
    $ip,
    'Plus de 10 requêtes en moins de 5 secondes',
    600
);
```

Bloquer une IP pendant une heure :

```php
Ip::bloquer(
    $ip,
    'Tentative de traversée de répertoire',
    3600
);
```

Bloquer une IP pendant 24 heures :

```php
Ip::bloquer(
    $ip,
    'Tentative répétée d\'injection SQL',
    86400
);
```

---

# 10. Journalisation automatique

`Ip::bloquer()` enregistre également le motif dans le journal des menaces :

```php
Journal::enregistrer($motif, 'menace');
```

Le développeur qui demande le blocage n'a donc pas besoin de journaliser lui-même la menace.

Il suffit de faire :

```php
Ip::bloquer(
    $ip,
    'Tentative de traversée de répertoire',
    3600
);
```

Le blocage et sa journalisation sont ainsi regroupés dans une seule opération.

---

# 11. Exemple avec `TraficIp`

La classe `TraficIp` détecte les comportements liés à un nombre excessif de requêtes.

Elle peut utiliser `Ip` pour effectuer le blocage :

```php
public static function surveiller(string $ip): bool
{
    self::enregistrerRequete($ip);

    if (self::estSuspect($ip)) {

        $motif =
            'Plus de 10 requêtes en moins de 5 secondes';

        Ip::bloquer($ip, $motif, 600);

        return true;
    }

    return false;
}
```

`TraficIp` est donc spécialisée dans la **détection**, tandis que `Ip` est spécialisée dans le **blocage**.

---

# 12. Exemple : traversée de répertoire

Une autre classe de sécurité peut détecter une tentative de traversée de répertoire.

Par exemple :

```php
if (ProtectionUrl::estSuspecte($url)) {

    Ip::bloquer(
        Ip::getIp(),
        'Tentative de traversée de répertoire',
        3600
    );

    throw new UserException(
        'Requête refusée.'
    );
}
```

La classe `Ip` n'a aucune connaissance de la traversée de répertoire.

Elle se contente d'enregistrer le blocage.

---

# 13. Exemple : tentative d'injection

Même principe pour une tentative d'injection :

```php
if (ProtectionInjection::estSuspecte($requete)) {

    Ip::bloquer(
        Ip::getIp(),
        'Tentative d\'injection détectée',
        3600
    );

    throw new UserException(
        'Requête refusée.'
    );
}
```

Cette architecture permet d'ajouter de nouveaux mécanismes de détection sans modifier la classe `Ip`.

---

# 14. Utilisation dans `bootstrap.php`

Le `bootstrap.php` constitue un emplacement adapté pour effectuer le contrôle général de l'adresse IP.

Exemple :

```php
$ip = Ip::getIp();

$blocage = Ip::getBlocage($ip);

if ($blocage !== null) {

    throw new UserException(
        'Votre adresse IP est temporairement bloquée pour le motif suivant : '
        . $blocage['motifblocage']
    );
}
```

Le contrôle est ainsi effectué avant de poursuivre le traitement de la requête.

Le principe devient :

```text
Requête HTTP
     │
     ▼
bootstrap.php
     │
     ▼
Ip::getIp()
     │
     ▼
Ip::getBlocage()
     │
     ├── blocage actif ──────► requête refusée
     │
     └── aucun blocage
              │
              ▼
       autres contrôles
              │
              ▼
       traitement de la requête
```

---

# 15. Consulter l'historique des blocages

La méthode `getAll()` permet de récupérer l'ensemble des blocages enregistrés :

```php
$blocages = Ip::getAll();
```

Chaque ligne contient :

```text
ip
dateblocage
motifblocage
dureeblocage
datefinblocage
```

Exemple :

```php
foreach (Ip::getAll() as $blocage) {

    echo $blocage['ip'];
    echo $blocage['motifblocage'];
    echo $blocage['dateblocage'];
    echo $blocage['datefinblocage'];
}
```

Cette méthode peut notamment être utilisée par une interface d'administration permettant de consulter les menaces détectées.

---

# 16. Résumé des méthodes publiques

|Méthode|Rôle|
|---|---|
|`Ip::getIp()`|Détermine l'adresse IP du client|
|`Ip::getBlocage($ip)`|Retourne les informations du blocage actif ou `null`|
|`Ip::estBloquee($ip)`|Indique si une IP est actuellement bloquée|
|`Ip::bloquer($ip, $motif, $duree)`|Crée un blocage et journalise la menace|
|`Ip::getAll()`|Retourne l'historique des blocages|

---

# 17. Principe à retenir

La classe `Ip` constitue le **point central de gestion des blocages IP** de l'application.

Les classes spécialisées détectent les comportements anormaux :

```text
TraficIp
ProtectionUrl
ProtectionInjection
...
```

mais délèguent le blocage à `Ip` :

```php
Ip::bloquer($ip, $motif, $duree);
```

La classe `Ip` centralise alors :

1. l'enregistrement du blocage ;
    
2. la conservation du motif ;
    
3. la conservation de la durée ;
    
4. la journalisation de la menace ;
    
5. la vérification des blocages actifs ;
    
6. la consultation de l'historique.
    

Cette organisation respecte le principe de **séparation des responsabilités** : les mécanismes de détection décident qu'un comportement est suspect, tandis que `Ip` assure la gestion technique du blocage.