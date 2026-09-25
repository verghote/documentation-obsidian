# Le serveur n'a pas renvoyé de réponse dans le format attendu (JSON).

Cette erreur survient lorsqu'un appel ajax réaliser à l'aide de la fonction appelAjax n'a pas renvoyé une réponse dans le format JSON
Cela se produit lorsqu'une erreur PHP 'non prévue' est renvoyée par le contrôleur (script PHP appelé)

Le contenu de cette erreur peut alors être consulté dans l'onglet console du navigateur (F12)

Exemple

```
( ! ) Warning: Constant DOSSIER_WWW already defined in J:\VirtualHostSlam\consultation\bootstrap\autoload.php on line 30
Call Stack
#TimeMemoryFunctionLocation
10.0235457528{main}(  )...\getlescoureurs.php:0
20.0259564304require( 'J:\VirtualHostSlam\consultation\bootstrap\autoload.php )...\getlescoureurs.php:13
30.0259564384define( $constant_name = 'DOSSIER_WWW', $value = 'J:\\VirtualHostSlam\\consultation\\public' )...\autoload.php:30
```

L'erreur ici vient du script lescoureurs.php qui a appelé 2 fois la même instruction : require $_SERVER['DOCUMENT_ROOT'] . "/../bootstrap/autoload.php";


# Une erreur technique est survenue, veuillez réessayer ultérieurement.

Une erreur a été interceptée et sa description exacte est stockée dans le fichier erreur.log du dossier log.

Selon le contexte d'exécution, l'erreur est soit affichée dans une boîte de dialogue lorsqu'elle est capturée par un appel Ajax, soit redirigée vers la page d'erreur lorsqu'elle est détectée lors d'un appel effectué directement depuis le navigateur

Exemple
02/08/2026 08:29:30 [Error] Class "Ajax" not found | code : 0 | fichier : J:\VirtualHostSlam\consultation\public\coureur\getbylicence\ajax\getbylicence.php | ligne : 13   /coureur/getbylicence/ajax/getbylicence.php    ::1

L'erreur vient du script getbylicence qui appelle une méthode de la classe Ajax  mais sans avoir ajouter l'instruction use ClasseTechnique\Ajax; au début du script

# Requête incorrecte

Il faut mettre un point d'arrêt sur la ligne en erreur dans la fonction appelAjax, relancer la page et regarder les lignes précédente pour découvrir l'erreur