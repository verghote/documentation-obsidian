
Lors de la création d'une source de données, PhpStorm télécharge automatiquement le pilote JDBC correspondant à la base de données.

Sur certains postes (proxy, pare-feu, restrictions réseau...), ce téléchargement peut échouer.

Dans ce cas, il est possible d'ajouter manuellement le pilote JDBC.

# 1. Télécharger le pilote JDBC

Télécharger le pilote JDBC correspondant à votre SGBD depuis le site officiel.

Pour MySQL (Connector/J), il s'agit d'un fichier de type :

```
mysql-connector-j-9.x.x.jar
```

# 2. Ouvrir les paramètres des sources de données

Dans PhpStorm :

```
View
    Tool Windows
        Database
```

Puis :

- cliquer sur **+**
- choisir **Data Source**
- sélectionner **MySQL**

# 3. Accéder aux pilotes

Dans la fenêtre **Data Sources and Drivers** :

- sélectionner l'onglet **Drivers** ;
- choisir le pilote **MySQL**.

# 4. Ajouter le fichier JAR

Dans la zone **Driver Files** :

- cliquer sur **+** ;
- choisir **Custom JARs...** ;
- sélectionner le fichier téléchargé :

```
mysql-connector-j-9.x.x.jar
```

PhpStorm détecte alors automatiquement le pilote JDBC.

# 5. Vérifier la classe du pilote

La classe du pilote doit être :

```
com.mysql.cj.jdbc.Driver
```

Si elle n'apparaît pas automatiquement, la saisir manuellement.

# 6. Tester la connexion

Revenir sur la source de données.

Cliquer sur :

```
Test Connection
```

Si tous les paramètres sont corrects, la connexion est établie.

# Remarque

L'ajout manuel du pilote n'est nécessaire qu'une seule fois.

Le pilote est ensuite disponible pour toutes les connexions MySQL créées dans PhpStorm.