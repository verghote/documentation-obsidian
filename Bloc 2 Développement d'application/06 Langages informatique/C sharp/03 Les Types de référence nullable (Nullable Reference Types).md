
Cette fonctionnalité est activée par défaut depuis .NET 6.

Par défaut, dans les versions récentes de C#, les types de référence comme les classes sont considérés comme **ne pouvant pas valoir `null`**.

Par exemple, si on essaye d'initialiser un objet d'une classe avec la valeur null, le compilateur lève un avertissement (le fameux **CS8600**) pour dire : 
_« Attention, vous essayez de mettre `null` dans une variable qui, par contrat, doit toujours contenir un vrai objet »_.

```C#
CompteBancaire c1 = null;
```

Pour éviter cet avertissement, plusieurs solutions sont possibles

# Solution A : ajouter  ? derrière le type de l'objet

```C#
CompteBancaire? c1 = null;
```

En ajoutant le **`?`** (`CompteBancaire?`), vous dites explicitement au compilateur :

> _« Je préviens mon code que cette variable **a le droit** de valoir `null` à un moment donné »_.

Le compilateur s'en servira ensuite pour vous forcer à faire des vérifications de sécurité (par exemple, vous avertir si vous essayez d'utiliser un objet sans vérifier s'il est `null` au préalable).

# Solution B : L'opérateur d'atténuation de null (`!`) — _L'opérateur "t'inquiète pas"_

Si vous êtes obligé d'initialiser une variable à `null` par défaut, mais que vous garantissez qu'elle sera instanciée plus loin dans le code avant d'être utilisée, vous pouvez utiliser le point d'exclamation (souvent appelé _null-forgiving operator_) :


```C#
CompteBancaire compteD = null!;
```

Cela force le compilateur à fermer les yeux et à accepter le `null` sans émettre d'avertissement. 

C'est utile dans certains cas précis, mais à utiliser avec modération car vous perdez la sécurité du compilateur.

# Solution C (déconseillée) : Désactiver les types de référence nullable 

Il est possible de désactiver complètement ce système de vérification stricte  pour tout le projet. 

Dans votre fichier de projet (`.csproj`), modifiez ou ajoutez ceci :

```XML
<PropertyGroup>
  <Nullable>disable</Nullable>
</PropertyGroup>
```

C# se comporte comme dans les anciennes versions (avant .NET 6) : n'importe quel objet peut valoir `null` par défaut, sans aucun avertissement.
 
 Il est possible de désactiver la règle pour un seul fichier

Si vous voulez garder la sécurité partout sauf dans un fichier spécifique, vous pouvez placer cette instruction tout en haut de votre script C# :

```C#
#nullable disable
```


# Choisir la meilleure solution

**La meilleure pratique moderne** est d'utiliser le **`?`** dès qu'une variable a le droit d'être vide (`null`), pour que le compilateur vous protège contre les erreurs de type `NullReferenceException`.
