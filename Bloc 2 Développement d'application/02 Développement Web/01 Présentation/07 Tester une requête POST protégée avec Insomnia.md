
Cette procédure permet de tester avec **Insomnia** une requête `POST` protégée par l'application.

Pour tester une requête POST protégée, il faut suivre cette séquence :

```text
1. Créer la requête POST
        ▼
2. Saisir l'URL
        ▼
3. Ajouter X-Requested-With
        ▼
4. Ouvrir le navigateur
        ▼
5. Récupérer le csrf-token
        ▼
6. Ajouter X-CSRF-Token dans Insomnia
        ▼
7. Dans le navigateur :
   Application → Cookies
        ▼
8. Récupérer PHPSESSID
        ▼
9. Ajouter PHPSESSID dans les Cookies d'Insomnia
        ▼
10. Configurer le Body POST
        ▼
11. Cliquer sur Send
        ▼
12. Analyser la réponse
```

Nous allons utiliser comme exemple le contrôleur :

```text
http://consultation/coureur/getbylicence/ajax/getbylicence.php
```

La requête sera construite progressivement jusqu'à pouvoir être envoyée au serveur.

# 1. Créer la requête POST

Dans **Insomnia**, créer une nouvelle requête.

Donner un nom à la requête, par exemple :

```text
Recherche coureur par licence
```

Choisir la méthode :

```text
POST
```

La requête doit donc commencer par :

```text
POST
```

# 2. Indiquer l'URL

Dans le champ URL d'Insomnia, saisir :

```text
http://consultation/coureur/getbylicence/ajax/getbylicence.php
```

On obtient :

```text
POST http://consultation/coureur/getbylicence/ajax/getbylicence.php
```

À ce stade, la requête existe, mais elle n'est pas encore prête à être envoyée.

# 3. Ajouter l'en-tête AJAX

L'application vérifie que le contrôleur est appelé comme un contrôleur AJAX.

Dans Insomnia, ouvrir l'onglet :

```text
Headers
```

Ajouter une ligne :

| Name             | Value          |
| ---------------- | -------------- |
| X-Requested-With | XMLHttpRequest |

La requête contient maintenant :

```http
X-Requested-With: XMLHttpRequest
```

Cet en-tête permet de reproduire le comportement d'une requête AJAX effectuée depuis le navigateur.

# 4. Récupérer le token CSRF dans le navigateur

Il faut maintenant récupérer le token CSRF utilisé par l'application.

Ouvrir dans le navigateur la page de l'application qui permet normalement d'effectuer la requête.

Ouvrir les outils de développement avec :

```text
F12
```

Dans le code HTML de la page, rechercher :

```html
<meta name="csrf-token" content="...">
```

Dans notre exemple :

```html
<meta name="csrf-token" content="0af803b66dbee97cc954aafd47c132476417cdefd9580e8e550a784ff1b94b97">
```

Le token est la valeur de l'attribut `content` :

```text
0af803b66dbee97cc954aafd47c132476417cdefd9580e8e550a784ff1b94b97
```

**Il faut toujours utiliser le token actuellement présent dans le navigateur.**

# 5. Ajouter le token CSRF dans Insomnia

Retourner dans Insomnia.

Dans l'onglet :

```text
Headers
```

ajouter une nouvelle ligne :

| Name         | Value                                                            |
| ------------ | ---------------------------------------------------------------- |
| X-CSRF-Token | 0af803b66dbee97cc954aafd47c132476417cdefd9580e8e550a784ff1b94b97 |

Dans notre exemple :

```text
X-CSRF-Token: 0af803b66dbee97cc954aafd47c132476417cdefd9580e8e550a784ff1b94b97
```

Les Headers de la requête sont maintenant :

| Name             | Value                                                            |
| ---------------- | ---------------------------------------------------------------- |
| X-Requested-With | XMLHttpRequest                                                   |
| X-CSRF-Token     | 0af803b66dbee97cc954aafd47c132476417cdefd9580e8e550a784ff1b94b97 |

# 6. Récupérer le PHPSESSID dans le navigateur

Le token CSRF est lié à la session PHP.

Il faut donc récupérer le cookie de session correspondant.

Dans le navigateur, ouvrir les outils de développement :

```text
F12
```

Puis sélectionner :

```text
Application
```

Dans le menu de gauche, rechercher :

```text
Cookies
```

puis sélectionner le domaine de l'application :

```text
http://consultation
```

La liste des cookies du domaine apparaît.

Rechercher le cookie :

```text
PHPSESSID
```

Dans notre exemple, sa valeur est :

```text
ijdo8nktkt23153gbpm790vs7j
```

Cette valeur est un exemple. Il faut utiliser **la valeur présente dans votre propre navigateur**.

# 7. Ajouter PHPSESSID dans Insomnia

Retourner dans Insomnia.

Le `PHPSESSID` est un **cookie**. Il ne doit donc pas être ajouté dans le Body et ce n'est pas un Header à saisir manuellement.

Dans Insomnia, utiliser le gestionnaire de **Cookies** de la requête.

À proximité du champ contenant l'URL de la requête, ouvrir la gestion des cookies.

Créer un cookie pour le domaine :

```text
consultation
```

avec :

| Name      | Value                      |
| --------- | -------------------------- |
| PHPSESSID | ijdo8nktkt23153gbpm790vs7j |

Insomnia ajoutera alors automatiquement ce cookie à la requête :

```http
Cookie: PHPSESSID=ijdo8nktkt23153gbpm790vs7j
```

La requête possède maintenant les deux éléments nécessaires à la session et à la protection CSRF :

```text
X-CSRF-Token
PHPSESSID
```

**Le `PHPSESSID` et le `X-CSRF-Token` doivent provenir de la même session du navigateur.**

# 8. Configurer le Body

La requête est maintenant correctement préparée au niveau des Headers et de la session.

Il faut ajouter les données que le contrôleur attend.

Dans Insomnia, ouvrir :

```text
Body
```

Puis choisir le format attendu par le contrôleur.

Par exemple, si le contrôleur attend des données de formulaire classiques, choisir :

```text
Form URL Encoded
```

Puis ajouter les paramètres nécessaires.

Par exemple :

| Key     | Value  |
| ------- | ------ |
| licence | 000010 |

# 9. Vérifier la requête

Avant de l'envoyer, la requête doit avoir la structure suivante :

### Méthode

```text
POST
```

### URL

```text
http://consultation/coureur/getbylicence/ajax/getbylicence.php
```

### Headers

```text
X-Requested-With: XMLHttpRequest
X-CSRF-Token: votre_token
```

### Cookie

```text
PHPSESSID=votre_session
```

### Body

Les données attendues par le contrôleur.

On peut représenter la requête ainsi :

```text
POST http://consultation/coureur/getbylicence/ajax/getbylicence.php

Headers
------------------------------------
X-Requested-With: XMLHttpRequest
X-CSRF-Token: 0af803b66d...

Cookie
------------------------------------
PHPSESSID=ijdo8nktkt23153gbpm790vs7j

Body
------------------------------------
licence=000010
```

# 10. Envoyer la requête

La requête est maintenant prête.

Cliquer sur :

```text
Send
```

Insomnia envoie alors la requête au contrôleur PHP.

Le serveur reçoit :

```text
POST
 │
 ├── Headers
 │     ├── X-Requested-With
 │     └── X-CSRF-Token
 │
 ├── Cookie
 │     └── PHPSESSID
 │
 └── Body
       └── données POST
```

---

# 11. Observer la réponse

Après l'envoi, Insomnia affiche la réponse du serveur.

Il faut notamment regarder :

- le code HTTP ;
- le contenu de la réponse ;
- les éventuelles erreurs ;
- les données JSON retournées.

Par exemple, une réponse peut être :

```json
{
	"licence": "000010",
	"nom": "JOLY",
	"prenom": "ALI",
	"sexe": "M",
	"dateNaissanceFr": "14/04/1982",
	"dateNaissance": "1982-04-14",
	"idCategorie": "M1",
	"nomClub": "ALBERT MEAULTE AEROSPA.AC",
	"idClub": "080028"
}
ou
{
	"error": "Le paramètre licence ne peut pas être vide."
}
ou 
{
	"errors": {
		"licence": "Numéro de licence invalide."
	}
}
ou 
{
	"errors": {
		"licence": "Numéro de licence inexistant."
	}
}
```

# 12. En cas d'erreur

Si le serveur refuse la requête, vérifier les éléments dans l'ordre.

### Le contrôleur refuse l'appel AJAX

Vérifier :

```text
X-Requested-With: XMLHttpRequest
```

### Le contrôleur refuse le token CSRF

Vérifier :

```text
X-CSRF-Token
```

et récupérer à nouveau le token dans la page du navigateur.

### Le contrôleur refuse la session

Vérifier :

```text
PHPSESSID
```

dans :

```text
Navigateur
→ F12
→ Application
→ Cookies
→ consultation
```

Puis vérifier que le même `PHPSESSID` est enregistré dans les cookies d'Insomnia.

### Le contrôleur refuse les données

Vérifier le :

```text
Body
```

et notamment le nom et la valeur des paramètres envoyés.


# Résumé

Pour tester une requête POST protégée, suivre exactement cette procédure :

```text
1. Créer la requête POST
          ↓
2. Indiquer l'URL
          ↓
3. Ajouter X-Requested-With: XMLHttpRequest
          ↓
4. Dans le navigateur :
   récupérer le csrf-token
          ↓
5. Ajouter X-CSRF-Token dans Insomnia
          ↓
6. Dans le navigateur :
   Application → Cookies
   récupérer PHPSESSID
          ↓
7. Dans Insomnia :
   ajouter PHPSESSID dans les Cookies
          ↓
8. Configurer le Body
          ↓
9. Envoyer la requête
          ↓
10. Observer la réponse
```

## Les trois endroits importants dans Insomnia

Au final, les informations sont réparties dans trois endroits différents :

```text
┌──────────────────────────────┐
│ Headers                      │
│                              │
│ X-Requested-With             │
│ X-CSRF-Token                 │
└──────────────────────────────┘

┌──────────────────────────────┐
│ Cookies                      │
│                              │
│ PHPSESSID                    │
└──────────────────────────────┘

┌──────────────────────────────┐
│ Body                         │
│                              │
│ Données envoyées en POST     │
└──────────────────────────────┘
```

Le point essentiel à retenir est que le **`X-CSRF-Token` et le `PHPSESSID` doivent provenir de la même session du navigateur**.