- **Injection SQL (SQLi)** : permet à un attaquant de manipuler une requête SQL afin d’accéder, modifier ou supprimer des données. La protection passe notamment par l’utilisation de **requêtes préparées**.
    
- **XSS (Cross-Site Scripting)** : consiste à injecter du code JavaScript malveillant dans une page Web afin qu’il soit exécuté dans le navigateur d’une victime. L’**échappement et la validation des données** permettent notamment de s’en protéger.
    
- **CSRF (Cross-Site Request Forgery)** : pousse un utilisateur authentifié à effectuer involontairement une action sur une application. L’utilisation de **tokens CSRF** constitue une protection courante.
    
- **Mauvaise gestion de l’authentification** : mots de passe mal protégés, sessions mal gérées ou mécanismes de connexion insuffisamment sécurisés peuvent permettre la compromission d’un compte.
    
- **Contrôle d’accès insuffisant** : un utilisateur peut accéder à des ressources ou effectuer des actions qui ne devraient pas être autorisées pour son niveau de privilège.
    
- **Upload de fichiers non sécurisé** : une application acceptant des fichiers sans contrôles suffisants peut permettre l’envoi de fichiers dangereux. Il faut notamment contrôler le **type, la taille, le nom et le stockage** des fichiers.
    
- **Exposition de données sensibles** : des informations confidentielles peuvent être accessibles ou transmises sans protection suffisante. Le **chiffrement**, HTTPS et une bonne gestion des secrets permettent de réduire ce risque.
    
- **Mauvaise configuration de sécurité** : mots de passe par défaut, messages d’erreur trop détaillés, services inutiles activés ou configuration incorrecte du serveur peuvent créer des failles.
    
- **Path Traversal** : permet de manipuler un chemin de fichier pour accéder à des fichiers qui ne devraient pas être accessibles.
    
- **Dépendances vulnérables** : une bibliothèque, un framework ou un composant utilisé par l’application peut contenir une vulnérabilité connue. Il est donc important de **maintenir les dépendances à jour**.