Ce document constitue un guide complet pour intégrer une base de données MySQL dans une application C# moderne, en s'appuyant sur le connecteur officiel et en structurant l'architecture logicielle autour d'une classe `Passerelle`.

## 1. Chargement et configuration du package NuGet

Pour un projet utilisant .NET, l'utilisation du connecteur officiel s'effectue via le package NuGet **`MySqlConnector`**.

- **Procédure dans Visual Studio :** Faites un clic droit sur votre projet > **Gérer les packages NuGet** > Recherchez et installez **`MySqlConnector`**.
    

> **Pourquoi `MySqlConnector` plutôt que `MySql.Data` ?**
> 
> L'ancien paquet `MySql.Data` (fourni historiquement par Oracle) est aujourd'hui considéré comme obsolète, car moins bien maintenu, moins performant et moins compatible avec les versions récentes de .NET (support asynchrone moins optimisé, etc.). `MySqlConnector` est le standard moderne recommandé.

## 2. Les 3 classes fondamentales du pilote

Le pilotage de la base de données repose principalement sur trois classes de base :

|**Classe**|**Rôle principal**|
|---|---|
|**`MySqlConnection`**|Permet de définir les caractéristiques de la connexion et d'ouvrir/fermer la liaison physique vers le serveur MySQL.|
|**`MySqlCommand`**|Permet de porter et d'exécuter des requêtes SQL (commandes textuelles ou procédures stockées) sur la base de données.|
|**`MySqlDataReader`**|Un curseur en lecture seule et orienté avant qui permet de parcourir efficacement le résultat d'une requête de type `SELECT` retournant plusieurs lignes.|

## 3. Référence technique des classes

### A. La classe `MySqlConnection`

|**Membre**|**Type**|**Rôle / Description**|
|---|---|---|
|**`MySqlConnection(chaine)`**|Constructeur|Initialise une connexion à l'aide d'une chaîne de connexion valide.|
|**`Open()`**|Méthode|Ouvre la liaison physique. Génère une exception `MySqlException` en cas d'échec.|
|**`Close()`**|Méthode|Ferme explicitement la connexion.|
|**`ConnectionString`**|Propriété|Obtient ou définit la chaîne de connexion associée.|

### B. La classe `MySqlCommand`

|**Membre**|**Type**|**Rôle / Description**|
|---|---|---|
|**`MySqlCommand()`**|Constructeur|Constructeur par défaut (les propriétés doivent être alimentées ensuite).|
|**`MySqlCommand(req, cnx)`**|Constructeur|Surcharge associant directement la requête SQL et l'objet de connexion.|
|**`ExecuteNonQuery()`**|Méthode|Exécute une requête de mise à jour (`INSERT`, `UPDATE`, `DELETE`). Retourne le nombre de lignes affectées.|
|**`ExecuteScalar()`**|Méthode|Exécute une requête ne retournant qu'une seule valeur (ex: `SELECT COUNT(*)`). Retourne un `object` à convertir.|
|**`ExecuteReader()`**|Méthode|Exécute une requête de sélection et retourne un `MySqlDataReader` (curseur).|
|**`CommandText`**|Propriété|Texte de la requête SQL, nom de table ou nom de procédure stockée.|
|**`CommandType`**|Propriété|Indique le type de commande (`CommandType.Text` ou `CommandType.StoredProcedure`).|
|**`Parameters`**|Propriété|Collection/dictionnaire des paramètres sécurisés de la commande.|

#### Méthodes et propriétés de la collection `Parameters` :

- **`AddWithValue("nom", valeur)`** : Ajoute un paramètre identifié par `@nom` et initialisé avec sa valeur.
    
- **`Add("nom", MySqlDbType, taille)`** : Déclare un paramètre typé (utile notamment pour les paramètres en sortie `out` ou les chaînes de caractères `VarChar` de taille spécifique).
    
- **`Clear()`** : Vide la collection des paramètres.
    
- **`Value`**, **`Direction`**, **`SqlDbType`**, **`Size`** : Propriétés d'un paramètre individuel pour configurer sa direction (`ParameterDirection.Input`, `Output`, etc.).
    

### C. La classe `MySqlDataReader` (Le curseur)

Permet de parcourir les enregistrements ligne par ligne.

|**Membre**|**Rôle / Description**|
|---|---|
|**`FieldCount`**|Nombre de colonnes présentes dans la ligne en cours.|
|**`Read()`**|Avance le curseur à l'enregistrement suivant. Retourne `false` s'il n'y a plus de lignes.|
|**`Close()`**|Ferme le lecteur de données et libère le flux.|
|**`GetString(nom)`, `GetInt32(nom)`, etc.**|Méthodes typées de récupération de données par nom de colonne (ou par index entier commençant à 0).|
|**`IsDBNull(index)`**|Retourne `true` si la colonne contient une valeur `NULL` en base de données. **Indispensable** pour éviter les plantages lors de la lecture de champs optionnels.|

## 4. Établir une connexion à la base de données

### Option A : Connexion par chaîne directe

C#

```
string chaineConnexion = "Server=localhost;Database=promotion;User Id=root;Password=";
MySqlConnection cnx = new MySqlConnection(chaineConnexion);

try {
    cnx.Open();
}
catch (MySqlException e) { 
    // e.Message contient le détail technique de l'erreur réseau ou d'authentification
    Console.WriteLine("Erreur : " + e.Message);
}
```

### Option B : Connexion sécurisée via le fichier de configuration (`appsettings.json`)

Dans les architectures professionnelles, la chaîne de connexion ne doit jamais être codée en dur dans le code source. On utilise le fichier `appsettings.json` combiné avec `Microsoft.Extensions.Configuration`.

1. **Le fichier `appsettings.json` :**
    
```
{
  "ConnectionStrings": {
    "MySQL": "Server=localhost;Database=promotion;User=root;Pwd="
  }
}
```

2. **Le chargement dans le code (Application Console ou Service) :**

```C#
IConfiguration configuration = new ConfigurationBuilder()
   .SetBasePath(AppContext.BaseDirectory)
   .AddJsonFile("appsettings.json", optional: false)
   .Build();

// .GetConnectionString() cherche automatiquement dans la section "ConnectionStrings"
string? chaine = configuration.GetConnectionString("MySQL");

MySqlConnection cnx = new MySqlConnection(chaine);
cnx.Open();
```

## 5. Exécution des requêtes SQL

### A. Requête retournant une seule valeur (`ExecuteScalar`)

```C#
cmd.CommandText = "SELECT nom FROM client WHERE id = 1;";
string unNom = (string) cmd.ExecuteScalar();

cmd.CommandText = "SELECT COUNT(*) FROM client;";
int nb = Int32.Parse(cmd.ExecuteScalar().ToString()!);

// Gestion sécurisée si le résultat peut être NULL
object valeur = cmd.ExecuteScalar();
if (valeur != null) {
    // Traitement
}
```

### B. Requête de sélection retournant plusieurs lignes (`ExecuteReader`)

```C#
List<Produit> lesProduits = new List<Produit>();
string sql = "SELECT id, nom, prix FROM produit WHERE idCategorie = @idCategorie";

using var cnx = new MySqlConnection(chaineConnexion);
cnx.Open();

using var cmd = new MySqlCommand(sql, cnx);
cmd.Parameters.AddWithValue("@idCategorie", idCategorie);

using var curseur = cmd.ExecuteReader();
while (curseur.Read()) {
    int id = curseur.GetInt32("id");
    string nom = curseur.GetString("nom");
    
    // Gestion propre des valeurs potentiellement NULL
    decimal prix = curseur.IsDBNull("prix") ? 0 : curseur.GetDecimal("prix");

    lesProduits.Add(new Produit(id, nom, prix));
}
```

### C. Requêtes de mise à jour (`ExecuteNonQuery`)

Pour les opérations `INSERT`, `UPDATE` ou `DELETE` sans paramètre :

```C#
string sql = "DELETE FROM produit WHERE id = 1;";
using var cmd = new MySqlCommand(sql, cnx);
cmd.ExecuteNonQuery();
```

Pour les requêtes paramétrées (pour éviter les injections SQL) :

```C#
string sql = "UPDATE produit SET prix = @prix WHERE id = @id";
using var cmd = new MySqlCommand(sql, cnx);
cmd.Parameters.AddWithValue("@prix", prix);
cmd.Parameters.AddWithValue("@id", code);
cmd.ExecuteNonQuery(); 
```

- **Réutilisation optimisée des paramètres pour les boucles d'insertion :**
    

```C#
sql = "INSERT INTO Produit (id, designation) VALUES (@id, @designation)";
using var cmd = new MySqlCommand(sql, cnx);
cmd.Parameters.Add("id", MySqlDbType.Int32);
cmd.Parameters.Add("designation", MySqlDbType.VarChar, 50);

foreach (Produit unProduit in lesProduits) {
    cmd.Parameters["id"].Value = unProduit.Reference;
    cmd.Parameters["designation"].Value = unProduit.Designation;
    cmd.ExecuteNonQuery();
}
```

### D. Récupération d'un ID auto-incrémenté après un INSERT

```C#
cmd.CommandText = "SELECT last_insert_id();";
object valeur = cmd.ExecuteScalar();
if (valeur != null) { 
    int dernierId = Int32.Parse(valeur.ToString()!); 
}
```

## 6. Utilisation des procédures et fonctions stockées

Les procédures stockées encapsulent la logique SQL directement sur le serveur SGBD.

### A. Appel d'une procédure stockée (Retour de curseur)

```C#
MySqlCommand cmd = new MySqlCommand() {
     Connection = cnx,
     CommandText = "getLesVisiteurs",
     CommandType = CommandType.StoredProcedure,
};

cmd.Parameters.AddWithValue("_idVisiteur", id);
using var curseur = cmd.ExecuteReader();
while (curseur.Read()) {
   string nom = curseur.GetString("nom");
}
```

### B. Appel d'une procédure avec paramètre en sortie (`OUT`)

```C#
MySqlCommand cmd = new MySqlCommand() {
    Connection = cnx,
    CommandText = "ajouterRendezVous",
    CommandType = CommandType.StoredProcedure,
};

cmd.Parameters.AddWithValue("_idPraticien", idPraticien);
cmd.Parameters.AddWithValue("_idMotif", idMotif);
cmd.Parameters.AddWithValue("_dateEtHeure", uneDate);

// Déclaration du paramètre de sortie OUT
cmd.Parameters.Add("_idVisite", MySqlDbType.Int32);
cmd.Parameters["_idVisite"].Direction = ParameterDirection.Output;

try {
    cmd.ExecuteNonQuery();
    int idVisite = (Int32)cmd.Parameters["_idVisite"].Value;
} catch (MySqlException e) {
    Console.WriteLine("Erreur SQL : " + e.Message);
}
```

## 7. Gestion des transactions

Une transaction garantit l'atomicité d'un ensemble d'opérations : **tout réussit, ou tout est annulé (`Rollback`)**.

```C#
cnx.Open();
MySqlTransaction transaction = cnx.BeginTransaction();

MySqlCommand cmd = new MySqlCommand() {
     Connection = cnx,
     Transaction = transaction,
     CommandType = CommandType.Text
};

try {
      cmd.CommandText = "INSERT INTO vente ...";
      cmd.ExecuteNonQuery();

      cmd.CommandText = "INSERT INTO detailvente ...";
      cmd.ExecuteNonQuery();

      // Validation définitive
      transaction.Commit();
} 
catch (MySqlException e) {
      // En cas d'erreur, annulation de toutes les modifications
      transaction.Rollback();
      Console.WriteLine("Erreur de transaction : " + e.Message);
}
cnx.Close();
```

## 8. L'architecture propre : La classe `Passerelle` (DAL)

Dans une application moderne, le code d'accès aux données ne doit pas être mélangé avec la logique métier ou l'interface graphique. La **`Passerelle`** (ou couche d'accès aux données - _DAL_) fait office de pont exclusif.

### Structure type de la classe `Passerelle` :

```C#
using Metier;
using Microsoft.Extensions.Configuration;
using MySqlConnector;

namespace Donnee;

public class Passerelle {
    private static readonly string chaineConnexion = ObtenirChaineConnexion();

    private static string ObtenirChaineConnexion() {
        IConfiguration configuration = new ConfigurationBuilder()
           .SetBasePath(AppContext.BaseDirectory)
           .AddJsonFile("appsettings.json", optional: false)
           .Build();
   
        string? chaine = configuration.GetConnectionString("MySQL");
        if (string.IsNullOrWhiteSpace(chaine)) {
            throw new InvalidOperationException("La chaîne de connexion 'MySQL' est absente ou vide.");
        }
        return chaine;
    }

    private static MySqlConnection ouvrirConnexion() {
        var cnx = new MySqlConnection(chaineConnexion);
        try {
            cnx.Open();
            return cnx;
        } catch (MySqlException e) {
            throw new Exception("Impossible d'ouvrir la connexion MySQL : " + e.Message, e);
        }
    }

    public static List<Etudiant> getLesEtudiants() {
        var liste = new List<Etudiant>();

        using var cnx = ouvrirConnexion();
        using var cmd = cnx.CreateCommand();
        cmd.CommandText = "SELECT nom, prenom, `option`, promotion FROM etudiant ORDER BY nom, prenom";

        using var curseur = cmd.ExecuteReader();
        while (curseur.Read()) {
            string nom = curseur.GetString("nom");
            string prenom = curseur.GetString("prenom");
            string option = curseur.GetString("option");
            int promotion = curseur.GetInt32("promotion");
            liste.Add(new Etudiant(nom, prenom, option, promotion));
        }
        return liste;
    }

    public static void InsererEtudiant(Etudiant e) {
        using var cnx = ouvrirConnexion();
        using var cmd = cnx.CreateCommand();

        cmd.CommandText = "INSERT INTO etudiant (nom, prenom, `option`, promotion) VALUES (@nom, @prenom, @option, @promotion)";
        cmd.Parameters.AddWithValue("@nom", e.Nom);
        cmd.Parameters.AddWithValue("@prenom", e.Prenom);
        cmd.Parameters.AddWithValue("@option", e.Option);
        cmd.Parameters.AddWithValue("@promotion", e.Promotion);

        cmd.ExecuteNonQuery();
    }
}
```