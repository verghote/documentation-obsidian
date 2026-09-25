Pour restreindre l'accès à une zone sensible d'un site web (ex. back-office ou espace d'administration), deux approches sont possibles :

- **Au niveau applicatif :** via un formulaire de connexion et une gestion des sessions en code (PHP, JS, etc.). Cette méthode est souple mais plus lourde à mettre en œuvre.
    
- **Au niveau serveur (Apache) :** en confiant l'authentification directement au serveur HTTP. Cette solution est extrêmement rapide à déployer et idéale lorsqu'il y a peu d'utilisateurs.
    

La protection par serveur repose sur la combinaison de deux fichiers :

1. **`.htaccess`** : placé dans le répertoire à protéger, il définit les règles d'accès et pointe vers le fichier des identifiants.
    
2. **`.htpasswd`** : contient la liste des paires `utilisateur:mot_de_passe`.

## 2. Accès et gestion de session

Lorsqu'un utilisateur tente d'accéder au répertoire protégé, le navigateur affiche une boîte de dialogue d'authentification HTTP.

### Comportement du navigateur

Une fois la connexion réussie, le navigateur met en cache les identifiants pour la session en cours. Pour forcer une déconnexion sans fermer le navigateur, deux méthodes existent :

- **Côté client :** vider l'historique et les données de connexion du navigateur (`Ctrl` + `Maj` + `Suppr`).
    
- **Côté serveur :** créer un fichier `deconnexion.php` dans le répertoire protégé pour envoyer un en-tête d'erreur HTTP 401 :
    

PHP

```
<?php
header('HTTP/1.0 401 Unauthorized');
?>
<meta http-equiv="refresh" content="0;url=/">
```

## 3. Création du fichier `.htaccess`

Le fichier `.htaccess` doit être placé à la racine du répertoire à sécuriser.

### Configuration type

Apache

```
AuthType Basic  
AuthUserFile "j:/VirtualHostSlam/ppe/public/backoffice/.htpasswd"  
AuthName "Administration"  
require valid-user  
  
# protection du fichier .htpasswd  
<Files ".htpasswd">  
    Require all denied  
</Files>
```

### Explication des directives

|**Directive**|**Description**|
|---|---|
|**`AuthType`**|Type d'authentification (`Basic` ou `Digest`).|
|**`AuthName`**|Intitulé affiché dans la boîte de dialogue de connexion.|
|**`AuthUserFile`**|**Chemin absolu** sur le disque vers le fichier `.htpasswd`. Une erreur dans le chemin provoque une erreur HTTP 500.|
|**`Require valid-user`**|Autorise tous les utilisateurs déclarés dans le fichier `.htpasswd`.|
|**`Require user user1`**|_(Optionnel)_ Autorise uniquement certains utilisateurs spécifiques.|

## 4. Configuration sous Windows (Version non hachée)

Sous un environnement local **Windows / WampServer**, Apache tolère l'utilisation de mots de passe en texte clair (Plaintext).

> ⚠️ **Avertissement :** Cette méthode est réservée exclusivement aux environnements de développement locaux sous Windows. Elle ne doit jamais être utilisée sur un serveur de production Linux.

### Mise en place rapide avec NotePad++

1. Créez un fichier nommé `.htpasswd` dans le répertoire de votre choix (idéalement hors de la racine web accessible publiquement).
    
2. Ajoutez les identifiants sous la forme `nom_utilisateur:mot_de_passe` (un couple par ligne).
    

**Exemple de contenu `.htpasswd` :**

Plaintext

```
admin:admin
```

## 5. Procédure pour obtenir un fichier `.htpasswd` haché

Sur un serveur de production (Linux) ou pour respecter les bonnes pratiques de sécurité, les mots de passe doivent être hachés. Apache prend en charge plusieurs formats de hachage (`bcrypt`, `MD5`, `SHA1`).

### Méthode A : Utilisation de l'outil en ligne de commande `htpasswd.exe`

L'utilitaire `htpasswd` est fourni directement avec Apache (ex. `C:\wamp64\bin\apache\apache2.4.65\bin\htpasswd.exe`).

#### 1. Création du fichier avec le premier utilisateur (`admin / admin`)

L'option `-c` crée le fichier, `-b` permet de passer le mot de passe dans la commande, et `-m` applique un hachage MD5 (ou `-B` pour du bcrypt).

Bash

```
# syntaxe
htpasswd -cbm ".htpasswd" admin admin
```

_Contenu généré dans `.htpasswd` :_

Plaintext

```
admin:$apr1$yx15Mgrx$aUduR6UcgvXRFlHxg1JJT1
```

#### 2. Ajout d'utilisateurs supplémentaires (`root / root`)

> **Attention :** N'utilisez plus l'option `-c`, sinon le fichier existant sera écrasé.

Bash

```
htpasswd -bm ".htpasswd" root root
```

_Contenu final du fichier `.htpasswd` :_

Plaintext

```
admin:$apr1$yx15Mgrx$aUduR6UcgvXRFlHxg1JJT1
root:$apr1$4PhDOg50$3wPtovZGp68hUl0OCAKQ20
```

### Méthode B : Génération via un script PHP (Format SHA1)

Il est possible de générer des lignes au format `{SHA}` sans utiliser l'invite de commande, en exécutant un court script PHP :

PHP

```php
<?php
$user = "admin";
$pass = "admin";

// Génération du condensé SHA-1 encodé en Base64
$hashed_password = "{SHA}" . base64_encode(sha1($pass, true));

// Format à insérer dans le fichier .htpasswd
echo $user . ":" . $hashed_password;
?>
```

**Résultat à copier dans le fichier `.htpasswd` :**

Plaintext

```
admin:{SHA}0DPiKuNIrrVmD8IUCuw1hQxNqZc=
```

## 6. Options de la commande `htpasswd`

Plaintext

```
htpasswd [-cimBdpsDv] [-C cost] fichier_mot_de_passe utilisateur
```

- **`-c`** : Crée un nouveau fichier `.htpasswd`.
    
- **`-b`** : Récupère le mot de passe depuis la ligne de commande.
    
- **`-m`** : Force le hachage en MD5 _(recommandé par défaut)_.
    
- **`-B`** : Force le hachage en Bcrypt _(très sécurisé)_.
    
- **`-D`** : Supprime l'utilisateur spécifié du fichier.
    
- **`-p`** : Ne chiffre pas le mot de passe (Plaintext - Windows uniquement).