pattern="^[A-Za-z0-9ÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸàâæçèéêëîïôœùûüÿ]+(?:[ '\-]?[A-Za-z0-9ÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸàâçèéêëîïôœùûüÿ]+)*$"

Pattern attribute value ^[A-Za-z0-9ÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸàâæçèéêëîïôœùûüÿ]+([ '-]?[A-Za-z0-9ÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸàâæçèéêëîïôœùûüÿ]+)*$ is not a valid regular expression: Uncaught SyntaxError: Failed to read the 'validationMessage' property from 'HTMLInputElement': Invalid regular expression: /^[A-Za-z0-9ÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸàâæçèéêëîïôœùûüÿ]+([ '-]?[A-Za-z0-9ÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸàâæçèéêëîïôœùûüÿ]+)*$/v: Invalid character in character class


Depuis les évolutions récentes de JavaScript, l'attribut HTML `pattern` est interprété avec le flag **`v`** (Unicode Sets), et ce mode impose des règles plus strictes dans les classes de caractères.

Le `-` doit être échappé dans une classe `[...]` avec le nouveau mode `v`.

Le navigateur considère déjà que le motif doit correspondre à l'ensemble de la valeur. donc ^  et $ sont inutiles

Version sous chrome 
pattern="[A-Za-z0-9ÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸàâæçèéêëîïôœùûüÿ]+(?:[ '\-]?[A-Za-z0-9ÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸàâçèéêëîïôœùûüÿ]+)*"
