Cette erreur survient généralement sous Windows lorsque le dossier se trouve sur une clé USB, un disque externe ou une partition (comme du FAT32/exFAT) qui ne gère pas les permissions d'utilisateurs Windows, ce qui pousse Git à bloquer l'accès par sécurité.

Pour débloquer la situation, il suffit d'exécuter la commande préconisée par Git dans votre terminal :

Plus radicalement en mode administrateur
git config --system --add safe.directory "*"