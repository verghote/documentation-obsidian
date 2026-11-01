
Source :
https://www.usine-digitale.fr/cybersecurite/identifiants-voles-protections-defaillantes-comment-les-lacunes-de-la-dgfip-ont-permis-a-un-hacker-de-siphonner-les-donnees-de-ses-usagers.L554QQ4X3NCF3EZCUUQJHR5Y4E.html?utm_source=newsletter&utm_medium=email&utm_campaign=info_ud-week-end&email=2647873&_emu=be6e60472056c6c1c86b1d8c23ebcdf0ec1978e1f8662449e8fbe2b354468435&src_email_send_date=2026-10-03&user_id_nl=2647873&idbdd=10589139&_ope=eyJndWlkIjoiYmU2ZTYwNDcyMDU2YzZjMWM4NmIxZDhjMjNlYmNkZjBlYzE5NzhlMWY4NjYyNDQ5ZThmYmUyYjM1NDQ2ODQzNSJ9

Résumé

Le piratage de la **DGFiP** n’a pas reposé sur une faille informatique spectaculaire. Le problème vient surtout de **l'accumulation de plusieurs faiblesses de sécurité**.

### 🔑 1. Le pirate a utilisé de vrais identifiants

Des identifiants et mots de passe d'agents de la DGFiP auraient été volés, vraisemblablement grâce à des **infostealers** (logiciels malveillants qui récupèrent les mots de passe sur les ordinateurs).

Pour certains accès importants, **un simple identifiant + mot de passe suffisait**, sans double authentification. VVie Publique+1

### 🖥️ 2. Des ordinateurs personnels auraient été impliqués

L'ANSSI estime que certains identifiants ont probablement été compromis sur des machines que la DGFiP ne contrôlait pas, notamment des **ordinateurs personnels d'agents**.

Le pirate pouvait donc disposer de comptes parfaitement légitimes sans avoir besoin de « casser » les systèmes.

### 🌐 3. Le réseau entre administrations a facilité la progression

L'attaquant aurait également compromis des infrastructures du ministère de l'Éducation nationale, puis utilisé le **Réseau interministériel de l'État (RIE)** pour atteindre des systèmes de la DGFiP.

Le problème relevé par l'ANSSI est notamment un **cloisonnement insuffisant** entre certains systèmes. VVie Publique

### 📥 4. Il a ensuite aspiré les données

Le pirate a utilisé une technique appelée **scraping** : un programme consulte automatiquement un très grand nombre de fiches et en récupère le contenu.

En juin, environ **11 Go de données** auraient été extraits, puis encore environ **3 Go en juillet**. Pourtant, ces extractions n'ont pas été détectées. Cclubic.com+1

### 🚨 5. C'est surtout la détection qui a échoué

C'est probablement le point le plus important de l'article.

Il existait pourtant plusieurs signaux suspects :

- connexions à des heures inhabituelles ;
    
- adresses IP suspectes ;
    
- connexions depuis l'étranger ;
    
- volumes importants de données ;
    
- nombre anormalement élevé de requêtes ;
    
- comptes déjà signalés comme potentiellement compromis.
    

Mais ces signaux n'ont pas été suffisamment **corrélés entre eux** pour déclencher une alerte efficace. VVie Publique+1

Et même lorsqu'un mot de passe était réinitialisé, **la session déjà ouverte pouvait continuer**.

### 👥 6. Combien de personnes sont concernées ?

La DGFiP a établi que des données concernant environ **678 000 particuliers et professionnels** avaient été consultées ou extraites.

Les informations pouvaient notamment comprendre :

- identité ;
    
- coordonnées ;
    
- identifiant fiscal ;
    
- situation familiale ;
    
- revenu fiscal de référence ;
    
- taux de prélèvement à la source ;
    
- échanges avec la DGFiP ;
    
- pour les entreprises, raison sociale et SIREN ;
    
- certaines données cadastrales. PPresse Économie+1
    

**Important : les espaces personnels impots.gouv.fr des contribuables n'ont pas été compromis et leurs mots de passe n'ont pas été volés dans cette attaque**, selon la DGFiP. PPresse Économie

## 🎯 Ce que l'ANSSI reproche essentiellement à la DGFiP

L'ANSSI résume les problèmes autour de **3 domaines** :

**1. Identité** → des comptes légitimes ont été compromis et certains accès ne nécessitaient pas de MFA.

**2. Architecture** → certains systèmes sensibles étaient insuffisamment cloisonnés.

**3. Détection** → les extractions massives de données n'ont pas été détectées à temps. VVie Publique

En clair, **le pirate n'a pas eu besoin de trouver une « porte secrète » dans le système : il a récupéré des clés légitimes, puis a pu se déplacer dans un environnement insuffisamment cloisonné et surveillé.**

### 🛡️ Les mesures envisagées

L'ANSSI recommande notamment :

- généraliser une **authentification multifacteur réellement indépendante** du mot de passe ;
    
- limiter l'utilisation d'ordinateurs personnels ;
    
- renforcer la surveillance des applications ;
    
- détecter les volumes de consultation anormaux ;
    
- couper les sessions lorsqu'un mot de passe est réinitialisé ;
    
- mieux filtrer les adresses IP suspectes ;
    
- limiter les données accessibles à chaque agent selon sa mission ;
    
- mieux cloisonner les réseaux entre administrations. VVie Publique
    

**La leçon principale :** ce n'est pas forcément une technologie extraordinairement sophistiquée qui a permis l'attaque, mais plutôt une chaîne de petites faiblesses — **identifiants volés + MFA insuffisant + réseau trop ouvert + surveillance insuffisante** — qui, mises bout à bout, ont permis l'exfiltration de centaines de milliers de dossiers. 

## 🔴 La chaîne d'attaque, étape par étape

On peut la résumer ainsi :

**1. Vol d'identifiants → 2. Connexion avec de vrais comptes → 3. Passage par le réseau interministériel → 4. Accès aux applications fiscales → 5. Scraping massif → 6. Détection trop tardive**

### 1. 🦠 Le pirate récupère les identifiants d'agents

Première étape : le pirate récupère les **identifiants et mots de passe de plusieurs dizaines d'agents de la DGFiP**.

L'ANSSI estime que cela est probablement lié à des **infostealers** : des malwares installés sur des ordinateurs qui récupèrent notamment les mots de passe enregistrés.

Le point particulièrement problématique est que certains agents utilisaient des **ordinateurs personnels** pour accéder aux ressources professionnelles. Ces machines échappaient donc au contrôle de sécurité de la DGFiP. CCyber Gouv

---

### 2. 🔑 Il n'a pas besoin de « hacker » le mot de passe

Le pirate possède maintenant un vrai :

```text
login : agent123
password : ********
```

Il peut donc se connecter comme un agent normal.

Et surtout, certains portails sensibles de la DGFiP **n'imposaient pas d'authentification forte**.

Autrement dit :

```text
Identifiant + mot de passe
          ↓
       ACCÈS
```

Il n'y avait pas systématiquement :

```text
Identifiant + mot de passe
          +
clé physique / application MFA
          ↓
       ACCÈS
```

L'ANSSI identifie explicitement cette faiblesse dans la gestion des identités. CCyber Gouv

---

### 3. 🌐 Le pirate entre par Internet

Il se connecte notamment au **PIGP**, un portail accessible depuis Internet.

Puis il utilise **ADER**, qui donne accès à d'autres applications du système fiscal.

Et là intervient un élément important : le **Réseau interministériel de l'État (RIE)**.

Le RIE permet aux différentes administrations de communiquer entre elles.

Le problème est que le cloisonnement n'était pas suffisant.

Schématiquement :

```text
Internet
   │
   ▼
[ PIGP ]
   │
   ▼
[ ADER ]
   │
   ▼
[ Réseau interministériel ]
   │
   ├── Éducation nationale
   │
   └── DGFiP
          │
          ▼
      Applications
          │
          ▼
       E-Contact
```

Le pirate avait notamment compromis des infrastructures de l'Éducation nationale et a pu utiliser cette position pour atteindre des ressources de la DGFiP via le réseau interministériel. Cclubic.com+1

---

### 4. 🚨 Et pourtant, il y avait déjà des signaux d'alerte

C'est probablement l'aspect le plus intéressant techniquement.

L'Éducation nationale avait transmis des **indicateurs techniques de compromission** aux autres ministères.

Certaines adresses IP utilisées par l'attaquant étaient donc déjà connues.

Mais le problème était notamment que les différentes informations n'étaient pas suffisamment **corrélées**.

Par exemple :

```text
Connexion depuis une IP suspecte
             +
Connexion à 4h du matin
             +
Compte compromis
             +
Nombre énorme de requêtes
             +
Volume inhabituel de données
             ↓
      🚨 DEVRAIT ÊTRE SUSPECT
```

Mais ces événements étaient essentiellement traités séparément.

---

### 5. 📥 Le pirate commence le « scraping »

C'est là que l'attaque devient particulièrement intéressante.

Il ne télécharge pas simplement un gros fichier contenant toutes les données.

Il utilise plutôt un programme automatisé qui fait quelque chose comme :

```text
ouvrir fiche 001
récupérer données
ouvrir fiche 002
récupérer données
ouvrir fiche 003
récupérer données
...
ouvrir fiche 500 000
récupérer données
```

C'est du **scraping**.

Le problème est que les comptes utilisés disposaient de suffisamment de droits pour consulter énormément de données.

Et l'application ne semblait pas avoir de mécanisme suffisamment efficace du type :

> « Cet utilisateur vient de consulter 100 000 fiches en quelques heures, c'est anormal. »

---

### 6. 📦 Des gigaoctets de données sortent

Les volumes sont impressionnants.

Selon le rapport présenté par l'ANSSI :

- environ **11 Go** de données ont transité entre le 22 et le 25 juin ;
    
- environ **3 Go** supplémentaires ont été extraits en juillet.
    

Et surtout, **ces exfiltrations n'ont pas été détectées au moment où elles se produisaient**. CCyber Gouv

C'est là qu'on voit la différence entre :

**« empêcher une intrusion »**

et

**« détecter quelqu'un qui utilise déjà un compte légitime »**.

Le deuxième problème était ici particulièrement important.

---

## 7. 🔄 Le changement de mot de passe ne suffisait même pas

Un élément assez surprenant ressort du rapport.

À un moment, un comportement suspect est détecté et le mot de passe du compte est réinitialisé.

Mais :

```text
Pirate connecté
       ↓
Mot de passe changé
       ↓
Session existante toujours active
       ↓
Pirate continue
       ↓
EXFILTRATION
```

La réinitialisation du mot de passe **ne coupait pas automatiquement la session déjà ouverte**.

L'ANSSI recommande donc notamment de faire en sorte qu'un changement de mot de passe entraîne la fermeture des sessions existantes. Cclubic.com

---

## 8. 👤 Pourquoi le pirate n'a-t-il pas été repéré ?

C'est là que toute l'attaque devient intéressante.

Le pirate n'était pas nécessairement :

> « connecté avec un compte pirate ».

Il était connecté avec :

> **un compte parfaitement légitime appartenant à un agent.**

Pour un système de sécurité qui regarde seulement :

```text
Utilisateur = Jean Dupont
Compte valide = OUI
Mot de passe valide = OUI
```

tout semble normal.

Il faut donc également regarder :

```text
Qui ?
Depuis où ?
À quelle heure ?
Sur quelle application ?
Combien de données ?
Combien de requêtes ?
Quel comportement habituel ?
Quelle adresse IP ?
Quel pays ?
```

C'est précisément le type de **corrélation comportementale** que l'ANSSI estime avoir fait défaut. CCyber Gouv

---

# 🧩 La faiblesse fondamentale

On pourrait représenter le problème comme ceci :

```text
              IDENTITÉ
                 │
       mot de passe compromis
                 │
                 ▼
           accès légitime
                 │
                 ▼
          RÉSEAU MAL CLOISONNÉ
                 │
                 ▼
       applications sensibles
                 │
                 ▼
          droits trop larges
                 │
                 ▼
          SCRAPING MASSIF
                 │
                 ▼
       DÉTECTION INSUFFISANTE
                 │
                 ▼
          données exfiltrées
```

**Aucune de ces faiblesses n'est nécessairement suffisante à elle seule.**

C'est leur combinaison qui permet l'attaque.

---

## 🎯 Et c'est ça que l'ANSSI reproche principalement

Le rapport officiel classe les problèmes en **trois grandes catégories** :

|Domaine|Problème|
|---|---|
|🔐 Identité|Identifiants volés + authentification forte insuffisante|
|🏗️ Architecture|Applications sensibles insuffisamment cloisonnées|
|👁️ Détection|Les comportements anormaux n'ont pas été correctement détectés|

CCyber Gouv+1

Et les données concernées étaient importantes : la DGFiP indique environ **678 000 particuliers et professionnels** concernés. Elle précise toutefois que **les espaces personnels impots.gouv.fr des usagers et leurs identifiants/mots de passe n'ont pas été compromis**. PPresse Économie+1

### 💡 La leçon pour une entreprise

Si tu travailles dans l'informatique, c'est probablement la partie la plus intéressante à retenir :

> **Le MFA ne suffit pas. Le cloisonnement ne suffit pas. Le SIEM ne suffit pas. Les antivirus ne suffisent pas.**

Il faut que les différentes couches travaillent ensemble :

```text
MFA
 ↓
Postes maîtrisés
 ↓
Zero Trust / segmentation
 ↓
Moindre privilège
 ↓
Logs applicatifs
 ↓
SIEM
 ↓
Détection comportementale
 ↓
Révocation des sessions
 ↓
Alertes SOC
```

Dans le cas de la DGFiP, **plusieurs de ces couches existaient**, mais elles n'étaient pas suffisamment efficaces ou coordonnées pour empêcher/détecter l'enchaînement complet. C'est précisément ce que l'ANSSI identifie dans son rapport.