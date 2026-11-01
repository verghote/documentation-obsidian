## 1. Qu'est-ce qu'un Package NuGet et pourquoi en a-t-on besoin ?

Dans le développement .NET moderne, vous ne réécrivez pas tout de zéro.

- **Un Package NuGet** est un composant logiciel tiers (prêt à l'emploi, packagé et versionné) que vous pouvez télécharger et intégrer dans votre application. Par exemple, `MySqlConnector` permet à votre code de dialoguer avec une base de données MySQL, et la famille `Microsoft.Extensions.Configuration` permet de lire les fichiers de configuration JSON. NuGet est le gestionnaire de paquets officiel de .NET (l'équivalent de _npm_ pour Node.js ou _pip_ pour Python).
    
- **Une Référence de Projet (`ProjectReference`)** permet de lier entre eux les différents projets de votre propre solution (par exemple, lier votre projet de gestion des données au projet `Metier`), afin qu'ils puissent communiquer et s'appeler mutuellement.
## 2. Méthode 1 : Via l'interface graphique de Visual Studio (Idéal pour débuter)

L'interface graphique de Visual Studio propose un gestionnaire visuel très pratique pour chercher et installer des packages sans toucher au code.

### A. Ajouter un Package NuGet

1. Dans l'**Explorateur de solutions**, faites un clic droit sur le **projet** (et non sur la solution globale) dans lequel vous souhaitez ajouter le package.
    
2. Cliquez sur **Gérer les packages NuGet...** (_Manage NuGet Packages..._).
    
3. Dans la fenêtre qui s'ouvre, cliquez sur l'onglet **Parcourir** (_Browse_).
    
4. Dans la barre de recherche en haut à gauche, tapez le nom du package recherché (ex: `MySqlConnector` ou `Microsoft.Extensions.Configuration.Json`).
    
5. Sélectionnez le package dans la liste.
    
6. Dans le volet de droite, vérifiez la version souhaitée et cliquez sur le bouton **Installer** (_Install_). Acceptez les éventuelles fenêtres de validation.

### B. Ajouter une Référence vers un autre projet local

1. Faites un clic droit sur votre **projet** cible dans l'Explorateur de solutions.
    
2. Survolez **Ajouter** (_Add_), puis cliquez sur **Référence de projet...** (_Project Reference..._).
    
3. Dans la fenêtre qui s'ouvre, cochez la case correspondant au projet que vous souhaitez lier (par exemple, le projet `Metier`).
    
4. Cliquez sur **OK**.
## 3. Méthode 2 : Directement dans le fichier `.csproj` (La méthode des développeurs expérimentés)

Les développeurs expérimentés privilégient souvent l'édition directe du fichier projet (`.csproj`). Pourquoi ? C'est **beaucoup plus rapide**, cela évite de cliquer dans des fenêtres modales, et c'est infiniment plus pratique pour copier-coller des configurations d'un projet à un autre ou travailler en équipe.

### Comment procéder ?

1. Dans l'**Explorateur de solutions**, faites un clic droit sur votre projet et choisissez **Modifier le fichier projet** (_Edit Project File_). Le fichier `.csproj` s'ouvre alors sous vos yeux au format XML.
    
2. Modifiez ou ajoutez directement vos balises `<PackageReference>` pour les packages NuGet et `<ProjectReference>` pour vos projets locaux à l'intérieur des blocs `<ItemGroup>` :

```XML
<Project Sdk="Microsoft.NET.Sdk">

	<PropertyGroup>
		<TargetFramework>net10.0</TargetFramework>
		<ImplicitUsings>enable</ImplicitUsings>
		<Nullable>enable</Nullable>
	</PropertyGroup>

	<!-- Déclaration des Packages NuGet externes -->
	<ItemGroup>
		<PackageReference Include="Microsoft.Extensions.Configuration" Version="10.0.0" />
		<PackageReference Include="Microsoft.Extensions.Configuration.FileExtensions" Version="10.0.0" />
		<PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="10.0.0" />
		<PackageReference Include="MySqlConnector" Version="2.6.2" />
	</ItemGroup>

	<!-- Déclaration des liaisons avec les autres projets locaux -->
	<ItemGroup>
		<ProjectReference Include="..\Metier\Metier.csproj" />
	</ItemGroup>

</Project>
```

3. **Sauvegardez le fichier** (`Ctrl + S`).
    
4. **Ce qui se passe automatiquement :** Dès la sauvegarde, Visual Studio détecte le changement, lance la **restauration NuGet** en arrière-plan, télécharge les packages nécessaires et les met à disposition de votre code. S'il subsiste un léger délai d'affichage, un simple clic droit sur le projet suivi de **Reconstruire** (_Rebuild_) remet tout en ordre instantanément.
# 4 Attention au package obsolète

Lorsqu'un développeur intègre un package tiers (comme `MySqlConnector`, `Newtonsoft.Json` ou des composants Entity Framework), il délégue une partie du code de l'application à un tiers.

Si une faille de sécurité est découverte dans ce code (par exemple, une mauvaise gestion du chiffrement TLS, une vulnérabilité de type injection, ou un contournement d'authentification), les éditeurs publient rapidement un correctif dans une nouvelle version du package.

- **Le danger :** Conserver une ancienne version (comme une version 2.x ancienne) expose l'application à des attaques automatisées que les pirates peuvent exploiter si l'application est connectée au réseau ou à Internet.
    
- **Le cas des connecteurs (ex. MySqlConnector) :** Les bibliothèques qui gèrent les protocoles réseaux et les bases de données sont particulièrement ciblées car elles manipulent des données sensibles (mots de passe, flux SQL, données personnelles). Des failles de type _Man-in-the-Middle_ sur les modes TLS ou des problèmes d'exposition de données ont déjà touché par le passé diverses versions de connecteurs SQL.

Les versions récentes de .NET et de Visual Studio intègrent nativement un outil appelé **NuGet Audit**.

À chaque fois que vous compilez votre projet ou que vous consultez vos packages, Visual Studio interroge une base de données publique des vulnérabilités connues (les CVE - _Common Vulnerabilities and Exposures_).

Si l'un de vos packages dans le fichier `.csproj` possède une faille de sécurité connue :

1. Visual Studio déclenche un **avertissement de sécurité** (Warning) dans la fenêtre _Liste d'erreurs_ (_Error List_).
    
2. Le code de l'avertissement commence généralement par `NU1901`, `NU1902`, `NU1903` ou `NU1904` selon la gravité de la faille (faible, modérée, critique).
    
Comment vérifier et mettre à jour vos packages dans Visual Studio ?

Pour vous prémunir de ces failles, voici la démarche à suivre :
### Via l'interface graphique :

1. Faites un clic droit sur votre **Solution** (ou votre projet) dans l'Explorateur de solutions.
    
2. Cliquez sur **Gérer les packages NuGet pour la solution...** (_Manage NuGet Packages for Solution_).
    
3. Cliquez sur l'onglet **Mises à jour** (_Updates_).
    
4. Visual Studio vous affichera la liste de tous les packages installés qui possèdent une version plus récente, en signalant ceux qui comportent des correctifs de sécurité.
    
5. Sélectionnez le package (ex: `MySqlConnector`), choisissez la dernière version stable, et cliquez sur **Mettre à jour** (_Update_).

### Via le fichier `.csproj` (La méthode directe) :

Il vous suffit de modifier directement le numéro de version dans vos balises `<PackageReference>` :

```XML
<ItemGroup>
  <!-- On passe d'une ancienne version vulnérable à une version récente et corrigée -->
  <PackageReference Include="MySqlConnector" Version="2.6.2" />
</ItemGroup>
```

En sauvegardant, Visual Studio télécharge automatiquement le correctif et referme la faille.

_C'est pour cette raison que dans les équipes professionnelles, les pipelines d'intégration continue (CI/CD) automatisent l'analyse des packages (avec des outils comme `dotnet list package --vulnerable`) pour bloquer la compilation ou le déploiement dès qu'une faille critique est détectée sur un composant._

# 5. Mais Attention aussi aux attaques sur la chaîne d'approvisionnement logiciel (software supply chain attacks)

Plutôt que d'attaquer directement un système bien protégé, des pirates choisissent de compromettre un maillon tiers de confiance comme un dépôt de packages.

Donc il faut aussi se montrer méfiant sur les mises à jour et ne jamais programmer de mise à jour automatique.

Voir [[Les attaques sur la chaîne d'approvisionnement logicielle (software supply chain attacks)]]
