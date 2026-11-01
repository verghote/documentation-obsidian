Message dans la console
 
 Deprecated: Function finfo_close() is deprecated since 8.5, as finfo objects are freed automatically in J:\VirtualHostSlam\upload\src\ClasseTechnique\InputFile.php on line <i>121</i>

Solution

Pour corriger cette dépréciation propre à PHP 8.5 (où les objets `finfo` sont désormais libérés automatiquement par le ramasse-miettes), il  suffit de **supprimer l'appel explicite à `finfo_close($finfo)`** à la ligne 121 de ton fichier `InputFile.php`.

Autres fonctions à supprimer
+ curl_close()
+ xml_parser_free()
+ openssl_free_key()
+ openssl_x509_free()