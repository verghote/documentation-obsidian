La classe `ClasseTechnique\Database` fournit un accès centralisé, sécurisé et optimisé à la base de données MySQL. Elle gère automatiquement l'instanciation de l'objet **`PDO`** en appliquant le design pattern **Singleton**.

##  Utilisation rapide

Pour exécuter des requêtes, il suffit d'appeler la méthode statique `getInstance()`. Elle retourne directement une instance **`PDO`** configurée.

PHP

```php
use ClasseTechnique\Database;

// 1. Récupération de l'instance PDO unique
$db = Database::getInstance();

// 2. Utilisation standard avec requêtes préparées
$cmd = $db->prepare("SELECT * FROM utilisateur WHERE id = :id");
$cmd->execute(['id' => $userId]);
$utilisateurs = $cmd->fetchAll();
```

## Fonctionnement & Caractéristiques

### 1. Pattern Singleton

- Vous n'avez pas besoin d'instancier la classe (`new Database()`).
    
- L'appel à `Database::getInstance()` vérifie si une connexion existe déjà :
    - Si **non** : charge la configuration, initialise la connexion et la stocke.
    - Si **oui** : réutilise la connexion existante.
        
- **Bénéfice :** Évite les connexions multiples inutiles à la base de données au cours d'un même script.
    
### 2. Configuration par défaut du moteur PDO

Une fois la connexion établie, les paramètres de sécurité et d'ergonomie suivants sont appliqués automatiquement :

| **Option / Commande**          | **Valeur / Effet**                        | **Description / Impact développeur**                                                                                                                  |
| ------------------------------ | ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PDO::ATTR_ERRMODE`            | `PDO::ERRMODE_EXCEPTION`                  | Les erreurs SQL déclenchent automatiquement des `PDOException`.                                                                                       |
| `PDO::ATTR_DEFAULT_FETCH_MODE` | `PDO::FETCH_ASSOC`                        | Les résultats des requêtes sont retournés sous forme de tableaux associatifs (ex: `$row['nom']`).                                                     |
| `PDO::ATTR_EMULATE_PREPARES`   | `false`                                   | Désactive l'émulation PHP des requêtes préparées et force les vraies requêtes préparées natives MySQL (sécurité renforcée contre les injections SQL). |
| `SET sql_mode`                 | `'STRICT_ALL_TABLES, ONLY_FULL_GROUP_BY'` | Active le mode strict MySQL (erreurs explicites sur données invalides, interdiction des ambiguïtés dans les `GROUP BY`).                              |

##  Configuration requise (`config/database.php`)

Les paramètres de connexion sont chargés depuis le fichier `config/database.php`.

### Structure du fichier de configuration :

PHP

```php
<?php
return [
    'host'     => 'localhost',
    'database' => 'consultation',
    'user'     => 'consultation',
    'password' => 'VotreMotDePasse',
    'port'     => 3306 // Optionnel (3306 par défaut si absent)
];
```

_(Le port est optionnel : s'il n'est pas défini, la valeur `3306` sera appliquée automatiquement)._

## ⚠️ Gestion des Erreurs & Exceptions

La méthode `getInstance()` peut lever deux types d'exceptions selon le problème rencontré :

1. **`RuntimeException`** :
    - Fichier `config/database.php` absent ou non lisible.
    - Format du fichier invalide (ne retourne pas un tableau).
    - Clé obligatoire manquante (`host`, `database`, `user` ou `password`).
        
2. **`PDOException`** :
    - Échec de la connexion à MySQL (ex: serveur indisponible, identifiants erronés, base inconnue).

Ces erreurs sont automatiquement traitées par la classe technique Erreur du Framework 
