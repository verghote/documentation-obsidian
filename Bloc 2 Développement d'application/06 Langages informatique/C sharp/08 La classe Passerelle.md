Dans une architecture logicielle moderne en C#, la séparation des responsabilités est un principe fondamental. La classe `Passerelle` joue un rôle clé dans cette organisation en faisant office de pont exclusif entre la base de données et l'application.

## 1. Positionnement dans l'architecture multicouche

Dans votre solution (composée notamment des projets `Donnee` et `Metier`), la classe `Passerelle` s'intercale précisément entre deux mondes :


```
[ Interface utilisateur / Application ] 
                 ↕ (Appelle)
         [ Projet Métier ]        <-- Contient la logique applicative et les entités (ex: Etudiant)
                 ↕ (Demande les données)
      [ Projet Données (Passerelle) ] <-- Fait le lien direct avec MySQL
                 ↕ (Requêtes SQL / Pilote MySqlConnector)
         [ Base de Données MySQL ]
```

- **Entre quoi et quoi ?** La passerelle est la passerelle (ou couche d'accès aux données - DAL) située **entre le code métier (les objets et règles de l'application)** et **le système de stockage externe (la base de données MySQL)**.
    
- **Son objectif principal :** Isoler totalement le code métier des détails techniques de la base de données. Le projet `Metier` ne sait pas ce qu'est MySQL, un connecteur, une requête SQL ou un curseur (`MySqlDataReader`). Il demande simplement des objets C# purs (`List<Etudiant>`), et c'est la `Passerelle` qui se charge de les récupérer et de les alimenter en mémoire.
    
## 2. Analyse de son contenu

La classe Passerelle prend en charge : 

1. **La centralisation de la chaîne de connexion** : Grâce au pattern `IConfiguration` et `ConfigurationBuilder()`, la passerelle lit dynamiquement le fichier `appsettings.json` au démarrage pour récupérer la configuration de la base sans rien coder en dur.

```C#
static private readonly string chaineConnexion = ObtenirChaineConnexion();

private static string ObtenirChaineConnexion() {
    IConfiguration configuration = new ConfigurationBuilder()
        .SetBasePath(AppContext.BaseDirectory)
        .AddJsonFile("appsettings.json", optional: false)
        .Build();

    // .GetConnectionString() cherche automatiquement dans la section "ConnectionStrings"
    string? chaine = configuration.GetConnectionString("MySQL");

    if (string.IsNullOrWhiteSpace(chaine)) {
        throw new InvalidOperationException("La chaîne de connexion 'MySQL' est absente ou vide dans appsettings.json.");
    }

    return chaine;
}
```


2. **L'ouverture sécurisée (`ouvrirConnexion`)** : Une méthode privée mutualise la création de l'objet `MySqlConnection` et intercepte les exceptions `MySqlException` pour renvoyer une erreur claire si le serveur est injoignable.

```C#
private static MySqlConnection ouvrirConnexion() {
    var cnx = new MySqlConnection(chaineConnexion);

    try {
        cnx.Open();
        return cnx;
    } catch (MySqlException e) {
        throw new Exception("Impossible d'ouvrir la connexion MySQL : " + e.Message, e);
    }
}
```

3. **Le chargement en mémoire 
    
    - Elle exécute les requêtes `SELECT` SQL permettant d'alimenter la structure de données en mémoire constituée de collections, de dictionnaires et autres objets,        
     permettant ainsi à l'application de travailler ensuite de manière autonome et rapide.

```C#
 public static List<Etudiant> getLesEtudiants() {
     var liste = new List<Etudiant>();

     using var cnx = ouvrirConnexion();
     using var cmd = cnx.CreateCommand();

     cmd.CommandText = "Select nom, prenom, `option`, promotion from etudiant order by nom, prenom";

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
```
        
4. **La prise en charge des opérations ciblées (CRUD : Create, Read, Update, Delete).

Exemple
```C#
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
```

```C#
public static void ModifierEtudiant(string ancienNom, string ancienPrenom, Etudiant nouvelEtudiant) {
    using var cnx = ouvrirConnexion();
    using var cmd = cnx.CreateCommand();

    cmd.CommandText = @"UPDATE etudiant 
                        SET nom = @nouveauNom, prenom = @nouveauPrenom, `option` = @option, promotion = @promotion 
                        WHERE nom = @ancienNom AND prenom = @ancienPrenom";

    cmd.Parameters.AddWithValue("@nouveauNom", nouvelEtudiant.Nom);
    cmd.Parameters.AddWithValue("@nouveauPrenom", nouvelEtudiant.Prenom);
    cmd.Parameters.AddWithValue("@option", nouvelEtudiant.Option);
    cmd.Parameters.AddWithValue("@promotion", nouvelEtudiant.Promotion);
    cmd.Parameters.AddWithValue("@ancienNom", ancienNom);
    cmd.Parameters.AddWithValue("@ancienPrenom", ancienPrenom);

    int lignesAffectees = cmd.ExecuteNonQuery();
    if (lignesAffectees == 0) {
        throw new InvalidOperationException("Aucun étudiant trouvé avec ces critères pour la modification.");
    }
}
```

```C#
public static void SupprimerEtudiant(string nom, string prenom) {
    using var cnx = ouvrirConnexion();
    using var cmd = cnx.CreateCommand();

    cmd.CommandText = "DELETE FROM etudiant WHERE nom = @nom AND prenom = @prenom";
    cmd.Parameters.AddWithValue("@nom", nom);
    cmd.Parameters.AddWithValue("@prenom", prenom);

    cmd.ExecuteNonQuery();
}
```

## 4. Bonnes pratiques indispensables intégrées

- **Utilisation de l'instruction `using`** : `using var cnx = ...` garantit que la connexion à la base de données et les commandes SQL sont proprement fermées et libérées de la mémoire dès qu'elles sortent du bloc de code, évitant ainsi les fuites de connexions (_connection leaks_).
    
- **Utilisation des paramètres (`Parameters.AddWithValue`)** : L'utilisation de paramètres nominatifs (`@nom`, `@prenom`) est **obligatoire** pour se prémunir totalement des attaques par **injection SQL**. On ne concatène jamais des variables directement dans du texte SQL.
    
- **Gestion des transactions (`BeginTransaction`)** : L'usage d'une transaction garantit qu'en cas d'erreur au milieu d'un traitement lourd, un `Rollback()` annule toutes les modifications pour laisser la base de données dans un état cohérent.