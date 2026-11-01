En ingénierie logicielle, l'organisation du code oppose fréquemment deux philosophies : la **vision théorique en couches** (issue des principes DDD, Clean Architecture ou Hexagonale) et la **vision pragmatique** (souvent guidée par le principe _KISS_ et le motif _Transaction Script_).

### 1. La vision théorique (Architecture en couches)

Cette approche découpe l'application en responsabilités strictes. Chaque couche ne communique qu'avec les couches adjacentes ou via des abstractions (interfaces), isolant totalement le code métier des détails d'infrastructure (base de données, système de fichiers, HTTP).

```
   ┌─────────────────────────────────────────┐
   │ 1. Couche Présentation / Contrôleurs    │  (Reçoit la requête, renvoie la réponse)
   └────────────────────┬────────────────────┘
                        │
   ┌────────────────────▼────────────────────┐
   │ 2. Couche Service / Application         │  (Orchestre les cas d'utilisation)
   └────────────────────┬────────────────────┘
                        │
   ┌────────────────────▼────────────────────┐
   │ 3. Couche Domaine / Métier              │  (Règles métier pures, entités)
   └────────────────────┬────────────────────┘
                        │
   ┌────────────────────▼────────────────────┐
   │ 4. Couche Infrastructure / Persistence  │  (Accès BDD, stockage fichiers, API tiers)
   └─────────────────────────────────────────┘
```

#### Principes clés

- **Inversion de dépendance :** Le domaine métier ne dépend d'aucune technologie d'infrastructure.
    
- **Orchestration isolée :** Les opérations impliquant plusieurs ressources (ex. BDD + Système de fichiers) sont encapsulées dans un **Service**.
    
- **Testabilité :** Chaque couche peut être testée unitairement en remplaçant l'infrastructure par des doublons de test (_mocks_).
    

#### Avantages

- **Pérennité :** Remplacer un composant (ex. changer de SGBD ou passer d'un stockage local à S3) n'impacte pas la logique métier.
    
- **Sécurité transactionnelle :** Les rollbacks inter-ressources (BDD + fichiers) sont centralisés et réutilisables.
    
- **Évolutivité d'équipe :** Plusieurs développeurs peuvent travailler sur des couches différentes sans conflit.
    

#### Inconvénients

- **Verbosité (_Boilerplate_) :** Multiplication du nombre de fichiers, d'interfaces et de classes passe-plat.
    
- **Complexité initiale :** Courbe d'apprentissage élevée et surcoût de développement au démarrage.
    

### 2. La vision pragmatique (KISS / Transaction Script)

Cette approche privilégie le chemin le plus court entre l'entrée (requête HTTP) et la sortie (réponse JSON ou HTML). Le contrôleur orchestre directement les briques techniques et les modèles nécessaires à la réalisation de l'action.

```
   ┌────────────────────────────────────────────────────────┐
   │ Contrôleur HTTP                                        │
   │                                                        │
   │  ├─► Valide l'entrée (InputFile)                       │
   │  ├─► Manipule le fichier (FileManager)                 │
   │  └─► Exécute la requête SQL (Document / BDD)           │
   └────────────────────────────────────────────────────────┘
```

#### Principes clés

- **Simplicité directe :** Le code est séquentiel, facile à lire de haut en bas sans indirection.
    
- **Moins de fichiers :** Absence d'interfaces ou de services intermédiaires tant que le besoin n'est pas avéré (_YAGNI — You Aren't Gonna Need It_).
    
- **Autonomie du script :** L'ensemble du flux d'une action réside à un seul endroit.
    

#### Avantages

- **Rapidité de développement :** Temps de mise en œuvre minimal.
    
- **Lisiabilité immédiate :** Un développeur comprend le flux complet en lisant un seul fichier.
    
- **Maintenance réduite sur petit périmètre :** Idéal pour les applications CRUD simples, scripts internes ou prototypes.
    

#### Inconvénients

- **Duplication de logique :** La gestion des erreurs ou des nettoyages (ex. annuler la copie si la BDD échoue) doit être répétée dans chaque contrôleur.
    
- **Couplage :** Réutiliser la logique métier hors du contexte Web (ex. via une commande CLI ou un CRON) nécessite de refactoriser ou de dupliquer du code.
    

### 3. Tableau comparatif

|**Critère**|**Vision Théorique**|**Vision Pragmatique**|
|---|---|---|
|**Objectif principal**|Maintenabilité & Découplage à long terme|Rapidité & Simplicité d'exécution|
|**Nombre de classes / fichiers**|Élevé (Contrôleur, Service, Repository, DTO...)|Faible (Contrôleur, Modèle, Utilities)|
|**Gestion des opérations complexes**|Centralisée dans la couche Application/Service|Directement dans le Contrôleur|
|**Facteur de décision**|Applications à forte logique métier ou évolutives|Applications CRUD, projets de taille moyenne, outils internes|

### 4. Syntaxe d'arbitrage : Quand choisir quelle approche ?

```
                            L'application doit-elle :
                            - Exécuter la logique via CLI/CRON/API ?
                            - Coordonner plusieurs ressources (BDD, FS, Mail) ?
                            - Évoluer sur plusieurs années avec une grande équipe ?
                                          │
                                ┌─────────┴─────────┐
                                │                   │
                               OUI                 NON
                                │                   │
                                ▼                   ▼
                      [Vision Théorique]   [Vision Pragmatique]
                      (Service / Couches)    (Direct / KISS)
```

#### Recommandation

1. **Démarrer pragmatique :** Ne créez pas de couches d'abstraction préventives. Si un contrôleur gère proprement une action simple, conservez cette structure.
    
2. **Refactoriser vers les services dès que :**
    
    - Une même orchestration (ex. `BDD + FileManager`) se répète dans plus de deux contrôleurs.
        
    - Vous devez exécuter cette action hors d'un contexte HTTP (commandes en ligne de commande, tâches planifiées).
        
    - La gestion des rollbacks (nettoyage en cas d'erreur) rend le contrôleur trop verbeux.