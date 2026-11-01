# 1. Présentation Théorique et Enjeux

## Pourquoi utiliser un fichier de configuration ?

Dans le développement professionnel d'applications, il est essentiel de séparer le **code source** des **données de configuration**. 

Les éléments suivants ne doivent **jamais** être écrits en dur (*hardcodés*) dans le code C# :
* Les chaînes de connexion aux bases de données (identifiants, mots de passe, serveurs).
* Les chemins d'accès vers les dossiers de stockage ou fichiers de données.
* Les clés d'API ou paramètres propres à l'environnement d'exécution.

## Les Avantages de cette approche :

1. **Portabilité et Déploiement :** Pour changer de base de données (passer du serveur de dev au serveur de production) ou modifier l'emplacement des fichiers de données, il suffit de modifier le fichier `.json`, sans récompiler le projet C#.
2. **Sécurité :** Évite de diffuser des mots de passe ou des chemins d'accès confidentiels dans le code source ou sur des dépôts de versioning (Git).
3. **Robustesse et Maintenance :** Centralise tous les réglages en un seul endroit.

# 2. Intégration dans Visual Studio (Pas à Pas)

## Étape 2.1 : Installation des packages NuGet

Pour utiliser la classe `ConfigurationBuilder` et lire les fichiers JSON sous .NET, vous devez ajouter les packages NuGet suivants à votre projet :
* `Microsoft.Extensions.Configuration`
* `Microsoft.Extensions.Configuration.Json`

*(Via Visual Studio : Clic droit sur le projet > **Gérer les packages NuGet...** > Chercher et installer ces deux packages).*

## Étape 2.2 : Créer le fichier `appsettings.json`

1. Dans l'**Explorateur de solutions**, faites un clic droit sur votre projet > **Ajouter** > **Nouvel élément...** (`Ctrl + Maj + A`).
2. Sélectionnez **Fichier JSON** (ou **Fichier texte** nommé `appsettings.json`).
3. Placez le fichier **à la racine de votre projet Interface** (à côté du fichier `.csproj`).

## Contenu du fichier `appsettings.json` :
```json
{
  "DataSettings": {
    "DataDirectory": "data"
  },
  "ConnectionStrings": {
    "MaConnexionBdd": "Server=localhost;Database=etudiants;User=root;Pwd="
  }
}
```

## Étape 2.3 : Créer le dossier `data` et ses fichiers

1. Clic droit sur le projet > **Ajouter** > **Nouveau dossier**, nommé `data`.
2. À l'intérieur de ce dossier, ajoutez votre fichier de données initial (ex: `etudiant.csv`).

## ⚠️ Étape 2.4 : La propriété "Copier dans le répertoire de sortie" (CRUCIAL)

Par défaut, Visual Studio ne copie pas les fichiers externes dans le répertoire d'exécution (`bin/Debug/netX.X/`). Si cette étape n'est pas faite, l'application ne trouvera ni `appsettings.json` ni vos fichiers de données.

Pour **`appsettings.json`** ET pour le fichier **`data/etudiant.csv`** :

1. Cliquez sur le fichier dans l'**Explorateur de solutions**.
2. Ouvrez la fenêtre **Propriétés** (`F4`).
3. Réglez la propriété **Copier dans le répertoire de sortie** sur :  
   👉 **Copier si plus récent** (*Copy if newer*).

# 3. Mise en Œuvre dans le Code C#

## 3.1. Classe utilitaire : `ConfigurationDonnees.cs` (Espace de noms `Donnee`)

Cette classe statique charge le fichier `appsettings.json` une seule fois au démarrage de la classe (via le champ `readonly`), puis calcule le chemin absolu du dossier `data`, qu'il soit exprimé en chemin relatif (`data`) ou absolu (`C:\Donnees`).

```csharp
using Microsoft.Extensions.Configuration;
using System;
using System.IO;

namespace Donnee 
{
    internal static class ConfigurationDonnees 
    {
        // Chargement unique au démarrage de la classe pour optimiser les performances
        private static readonly string dossierData = ObtenirDossierData();

        private static string ObtenirDossierData() 
        {
            // Initialisation du reader de configuration
            IConfiguration configuration = new ConfigurationBuilder()
                .SetBasePath(AppContext.BaseDirectory)
                .AddJsonFile("appsettings.json", optional: false)
                .Build();

            // Lecture directe de la clé dans la section DataSettings
            string dossier = configuration["DataSettings:DataDirectory"]!;

            // Résolution hybride : Absolu ou Relatif par rapport au dossier d'exécution
            return Path.IsPathRooted(dossier)
                ? dossier
                : Path.GetFullPath(Path.Combine(AppContext.BaseDirectory, dossier));
        }

        /// <summary>
        /// Renseigne le chemin absolu complet vers un fichier situé dans le dossier de données.
        /// </summary>
        public static string ObtenirCheminFichier(string nomFichier) 
        {
            return Path.Combine(dossierData, nomFichier);
        }
    }
}
```

- **En mode local (Développement) :** On utilise des chemins relatifs par rapport au dossier d'exécution (comme `./Data`), grâce à la copie automatique des fichiers dans le dossier de sortie (ce qu'on s'est dit juste avant). C'est pratique pour coder sur son poste.
    
- **En mode professionnel / Production (Serveur d'entreprise, Cloud, Application installée) :** Sur un serveur de production, les fichiers de données ou de configuration ne se trouvent _jamais_ à côté de l'exécutable (pour des raisons de sécurité, de droits d'écriture et de sauvegarde).
    
    - Le fichier `appsettings.json` indique alors un **chemin absolu standardisé** propre au système d'exploitation cible, par exemple :
        
        - Sous Windows (Service Windows ou IIS) : `C:\ProgramData\MonApplication\Data` ou `D:\DonneesApp\`
            
        - Sous Linux (Docker ou serveur Linux) : `/var/lib/monapplication/data`

## 3. 2 Pourquoi passer par un Objet IConfiguration et un Objet ConfigurationBuilder ?

À première vue, pour lire un simple chemin dans un fichier JSON, on pourrait se dire qu'un simple `File.ReadAllText("appsettings.json")` combiné avec un petit parseur JSON fait l'affaire. Alors, pourquoi tout ce mécanisme ?

`IConfiguration` est **une interface standardisée de .NET** (provenant du package `Microsoft.Extensions.Configuration`). Elle représente un dictionnaire unifié de paires **clé/valeur** (ex: `DataSettings:DataDirectory` -> `data`).

Peu importe d'où viennent les données (un fichier JSON, un fichier XML, les variables d'environnement du système, ou les arguments de la ligne de commande), `IConfiguration` présente toujours le même visage à votre code : vous lui demandez une clé, il vous renvoie la valeur.

Même pour un simple fichier de configuration, `IConfiguration` apporte des avantages majeurs qu'un code maison n'aurait pas :

1. **L'abstraction des sources (Le gros avantage professionnel) :** Dans une application pro, les paramètres ne viennent pas toujours d'un fichier JSON. Par exemple, sur un serveur cloud ou dans Docker, on modifie souvent la configuration via des **variables d'environnement**. Si vous aviez écrit un code personnalisé lisant le JSON directement, vous seriez bloqué pour lire des variables système. Avec `IConfiguration`, vous pouvez ajouter une ligne pour lire aussi l'environnement :
    
   
    ```C#
    IConfiguration configuration = new ConfigurationBuilder()
        .SetBasePath(AppContext.BaseDirectory)
        .AddJsonFile("appsettings.json", optional: false)
        .AddEnvironmentVariables() // <-- Lit aussi les variables du système d'exploitation !
        .Build();
    ```
    
    Si la variable d'environnement `DataSettings__DataDirectory` existe sur le serveur, .NET l'utilisera automatiquement **sans que vous n'ayez à modifier une seule ligne de code C#**.
    
1. **La gestion automatique des hiérarchies (Sections) :** Votre JSON contient des objets imbriqués (`DataSettings` -> `DataDirectory`).  IConfiguration sait parser nativement ces structures grâce au caractère deux-points `:` (`configuration["DataSettings:DataDirectory"]`). Écrire un parseur récursif pour gérer cela à la main demanderait beaucoup de code.
    
2. **La robustesse et le standard de l'écosystème .NET :** C'est la brique officielle utilisée dans tous les types d'applications .NET (Web API, applications de bureau WPF/WinForms, Services Windows, etc.). N'importe quel développeur .NET qui arrive sur votre projet reconnaîtra immédiatement ce pattern.


Pour bien comprendre ce bloc de code, il faut savoir qu'il utilise un design pattern très populaire en programmation : le **Builder Pattern** (ou patron de conception "Monteur").

L'idée est d'utiliser un "constructeur" intermédiaire qui assemble une configuration étape par étape, enchaînant les méthodes (on appelle cela le _method chaining_ ou chaînage de méthodes), jusqu'à obtenir l'objet final désiré.

Voici le rôle exact de chaque ligne de cette séquence :

### 1. `new ConfigurationBuilder()`

- **Son rôle :** C'est le point de départ, le "chef d'orchestre".
    
- **Explication :** Il instancie un objet vide de type `ConfigurationBuilder`. Cet objet a pour unique mission de collecter et d'empiler différentes sources de configuration (fichiers JSON, variables d'environnement, arguments de la ligne de commande, etc.) avant de les fusionner.
    

### 2. `.SetBasePath(AppContext.BaseDirectory)`

- **Son rôle :** Indiquer au constructeur **où** il doit chercher les fichiers.
    
- **Explication :** Par défaut, si vous lui dites de chercher `"appsettings.json"`, il va regarder dans le dossier courant du système d'exploitation (qui peut changer selon la façon dont l'application est lancée). En lui passant `AppContext.BaseDirectory`, vous lui donnez l'ordre strict de chercher dans le répertoire d'exécution de l'application (le dossier `bin/Debug/...` ou le dossier d'installation), ce qui garantit qu'il trouvera le fichier peu importe d'où l'utilisateur double-clique sur l'exécutable.
    

### 3. `.AddJsonFile("appsettings.json", optional: false)`

- **Son rôle :** Ajouter un fichier JSON spécifique à la liste des sources à lire.
    
- **Ses arguments :**
    
    - `"appsettings.json"` : Le nom du fichier à charger.
        
    - `optional: false` : C'est une règle de sécurité cruciale. Si `optional` est réglé sur `false`, cela signifie que **le fichier est obligatoire**. Si le fichier est absent au moment de lancer l'application, .NET planquera immédiatement avec une erreur explicite. Si vous aviez mis `true`, l'application aurait démarré sans fichier, quitte à planter plus tard de manière imprévisible.

### 4. `.Build()`

- **Son rôle :** Finaliser l'assemblage et générer l'objet prêt à l'emploi.
    
- **Explication :** C'est l'étape finale. Jusqu'à présent, le `ConfigurationBuilder` ne faisait que lister les instructions ("je veux chercher ici, je veux lire ce fichier..."). Lorsque vous appelez `.Build()`, il exécute réellement le travail : il ouvre le fichier JSON, analyse sa structure, lit son contenu, et retourne l'objet `IConfiguration` finalisé, structuré et prêt à vous restituer vos clés (comme `"DataSettings:DataDirectory"` ou `"ConnectionStrings:MaConnexionBdd"`).
### En résumé

Ce bloc signifie :

> _"Crée-moi un gestionnaire de configuration, prends pour base le dossier d'exécution de l'application, charge-y le fichier obligatoire `appsettings.json`, et construis-moi l'objet final !"_
## 3.3. Exploitation pour les fichiers : `Csv.cs`

La classe `Csv` s'appuie directement sur `ConfigurationDonnees` pour déterminer l'emplacement de `etudiant.csv` sans se soucier de l'arborescence physique sous-jacente.

    private static readonly string cheminFichier = ConfigurationDonnees.ObtenirCheminFichier("etudiant.csv");


```csharp
using System;
using System.Collections.Generic;
using System.IO;
using Donnee;

public class Csv 
{
    // Récupération dynamique du chemin du fichier
    private static readonly string cheminFichier = ConfigurationDonnees.ObtenirCheminFichier("etudiant.csv");

    public static List<Etudiant> getLesEtudiants() 
    {
        List<Etudiant> liste = new();

        if (File.Exists(cheminFichier)) 
        {
            string[] lesLignes = File.ReadAllLines(cheminFichier);
            foreach (string ligne in lesLignes) 
            {
                string[] lesColonnes = ligne.Split(';');
                string nom = lesColonnes[0];
                string prenom = lesColonnes[1];
                string option = lesColonnes[2];
                int promotion = Int32.Parse(lesColonnes[3]);

                liste.Add(new Etudiant(nom, prenom, option, promotion));
            }
        }

        return liste;
    }

    public static void sauver(List<Etudiant> liste) 
    {
        using StreamWriter fichier = new(cheminFichier, false);
        foreach (Etudiant e in liste) 
        {
            string ligne = $"{e.Nom};{e.Prenom};{e.Option};{e.Promotion}";
            fichier.WriteLine(ligne);
        }
    }
}
```


## 3.4. Exploitation pour la Base de Données : `Passerelle.cs`

Pour la base de données, on utilise la méthode d'extension standard `.GetConnectionString("MaConnexionBdd")`, qui cherche automatiquement dans la section `"ConnectionStrings"` du fichier JSON.

```csharp
using Microsoft.Extensions.Configuration;
using MySql.Data.MySqlClient;
using System;

public class Passerelle 
{
    private static readonly string chaineConnexion = ObtenirChaineConnexion();

    private static string ObtenirChaineConnexion() 
    {
        IConfiguration configuration = new ConfigurationBuilder()
            .SetBasePath(AppContext.BaseDirectory)
            .AddJsonFile("appsettings.json", optional: false)
            .Build();

        // Récupération automatique depuis la section "ConnectionStrings"
        string? cnx = configuration.GetConnectionString("MaConnexionBdd");

        if (string.IsNullOrWhiteSpace(cnx)) 
        {
            throw new InvalidOperationException("La chaîne de connexion 'MaConnexionBdd' est absente ou vide dans appsettings.json.");
        }

        return cnx;
    }

    private static MySqlConnection ouvrirConnexion() 
    {
        var cnx = new MySqlConnection(chaineConnexion);

        try 
        {
            cnx.Open();
            return cnx;
        }
        catch (MySqlException e) 
        {
            throw new Exception("Impossible d'ouvrir la connexion MySQL : " + e.Message, e);
        }
    }
}
```


# 4. Synthèse du Fonctionnement à l'Exécution

```
Projet C# (Source)
├── appsettings.json ------------(Copier si plus récent)----┐
├── data/                                                     │
│   └── etudiant.csv ------------(Copier si plus récent)----┼─> bin/Debug/netX.X/ (Répertoire d'exécution)
└── Code C# (ConfigurationDonnees, Csv, Passerelle) ---------┘    ├── appsettings.json
                                                                  ├── data/
                                                                  │   └── etudiant.csv
                                                                  └── MonApplication.exe
```

1. Lors de la **compilation**, Visual Studio duplique l'arborescence des fichiers marqués en "Copier si plus récent" vers le répertoire d'exécution `bin/Debug/...`.
2. Lors de l'**exécution**, `AppContext.BaseDirectory` pointe vers ce répertoire `bin/Debug/...`.
3. La classe `ConfigurationDonnees` lit la valeur `"data"`, la combine avec le répertoire d'exécution et fournit des chemins absolus parfaits et sans risque d'erreur au reste de l'application.