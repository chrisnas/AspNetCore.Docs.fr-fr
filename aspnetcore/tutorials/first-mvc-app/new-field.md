---
title: Ajouter un nouveau champ à une application ASP.NET Core MVC
author: rick-anderson
description: Découvrez comment utiliser la fonctionnalité Migrations Code First d’Entity Framework pour ajouter un nouveau champ à un modèle et migrer ce changement vers une base de données.
ms.author: riande
ms.custom: mvc
ms.date: 12/13/2018
uid: tutorials/first-mvc-app/new-field
ms.openlocfilehash: 6a2a2ca45f793ab95d45281ebb23180ac64761ec
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/18/2019
ms.locfileid: "71082315"
---
# <a name="add-a-new-field-to-an-aspnet-core-mvc-app"></a>Ajouter un nouveau champ à une application ASP.NET Core MVC

Par [Rick Anderson](https://twitter.com/RickAndMSFT)

Dans cette section, Migrations [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First est utilisé pour :

* Ajouter un nouveau champ au modèle.
* Migrer le nouveau champ vers la base de données.

Quand EF Code First est utilisé pour créer automatiquement une base de données, Code First :

* Ajoute une table à la base de données pour en suivre le schéma.
* Vérifie que la base de données est synchronisée avec les classes de modèle à partir desquelles elle a été générée. S’ils ne sont pas synchronisés, EF lève une exception. Cela facilite la recherche de problèmes d’incohérence de code/de bases de données.

## <a name="add-a-rating-property-to-the-movie-model"></a>Ajouter une propriété Rating au modèle Movie

Ajouter une propriété `Rating` à *Models/Movie.cs* :

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Models/MovieDateRating.cs?highlight=13&name=snippet)]

Générer l’application

### <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

 Ctrl+Maj+B

### <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
dotnet build
```

### <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

Commande ⌘ + B

------

Comme vous avez ajouté un nouveau champ à la classe `Movie`, vous devez mettre à jour la liste verte des liaisons pour y inclure cette nouvelle propriété. Dans *MoviesController.cs*, mettez à jour l’attribut `[Bind]` des méthodes d’action `Create` et `Edit` pour y inclure la propriété `Rating` :

```csharp
[Bind("Id,Title,ReleaseDate,Genre,Price,Rating")]
   ```

Mettez à jour les modèles de vue pour afficher, créer et modifier la nouvelle propriété `Rating` dans la vue du navigateur.

Ouvrez le fichier */Views/Movies/Index.cshtml* et ajoutez un champ `Rating` :

[!code-HTML[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexGenreRating.cshtml?highlight=16,38&range=24-64)]

Mettez à jour le fichier */Views/Movies/Create.cshtml* avec un champ `Rating`.

# <a name="visual-studio--visual-studio-for-mactabvisual-studiovisual-studio-mac"></a>[Visual Studio / Visual Studio pour Mac](#tab/visual-studio+visual-studio-mac)

Vous pouvez copier/coller le « groupe de formulaire » précédent et laisser IntelliSense vous aider à mettre à jour les champs. IntelliSense fonctionne avec des [Tag Helpers](xref:mvc/views/tag-helpers/intro).

![Le développeur a tapé la lettre R comme valeur d’attribut asp-for dans le deuxième élément étiquette de la vue. Un menu contextuel IntelliSense s’est affiché, montrant les champs disponibles, notamment Rating, qui est automatiquement mis en surbrillance dans la liste. Quand le développeur clique sur le champ ou appuie sur Entrée, la valeur est définie sur Rating.](new-field/_static/cr.png)

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!-- This tab intentionally left blank. -->

---

Mettez à jour les modèles restants.

Mettez à jour la classe `SeedData` pour qu’elle fournisse une valeur pour la nouvelle colonne. Vous pouvez voir un exemple de modification ci-dessous, mais elle doit être appliquée à chaque `new Movie`.

[!code-csharp[](start-mvc/sample/MvcMovie/Models/SeedDataRating.cs?name=snippet1&highlight=6)]

L’application ne fonctionne pas tant que la base de données n’est pas mise à jour pour inclure le nouveau champ. Si elle est exécutée maintenant, l’erreur `SqlException` est levée :

`SqlException: Invalid column name 'Rating'.`

Cette erreur survient car la classe du modèle Movie mise à jour est différente du schéma de la table Movie de la base de données existante. (Il n’existe pas de colonne `Rating` dans la table de base de données.)

Plusieurs approches sont possibles pour résoudre l’erreur :

1. Laissez Entity Framework supprimer et recréer automatiquement la base de données sur la base du nouveau schéma de classe de modèle. Cette approche est très utile au début du cycle de développement quand vous effectuez un développement actif sur une base de données de test. Elle permet de faire évoluer rapidement le schéma de modèle et de base de données ensemble. L’inconvénient, cependant, est que vous perdez les données existantes dans la base de données. Par conséquent, n’utilisez pas cette approche sur une base de données de production. L’utilisation d’un initialiseur pour amorcer automatiquement une base de données avec des données de test est souvent un moyen efficace pour développer une application. Il s’agit d’une bonne approche pour le développement initial et lors de l’utilisation de SQLite.

2. Modifier explicitement le schéma de la base de données existante pour le faire correspondre aux classes du modèle. L’avantage de cette approche est que vous conservez vos données. Vous pouvez apporter cette modification manuellement ou en créant un script de modification de la base de données.

3. Utilisez les migrations Code First pour mettre à jour le schéma de base de données.

Pour ce didacticiel, les migrations Code First sont utilisées.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Dans le menu **Outils**, sélectionnez **Gestionnaire de package NuGet > Console du Gestionnaire de package**.

  ![Menu Console du Gestionnaire de package](adding-model/_static/pmc.png)

Dans la console du Gestionnaire de package, entrez les commandes suivantes :

```powershell
Add-Migration Rating
Update-Database
```

La commande `Add-Migration` indique au framework de migration d’examiner le modèle `Movie` actuel avec le schéma de la base de données `Movie` actuel et de créer le code nécessaire pour migrer la base de données vers le nouveau modèle.

Le nom « Rating » est arbitraire et est utilisé pour nommer le fichier de migration. Il est utile d’utiliser un nom explicite pour le fichier de migration.

Si tous les enregistrements de la base de données sont supprimés, la méthode d’initialisation amorce la base de données et inclut le champ `Rating`.

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

Supprimer la base de données et utiliser des migrations pour recréer la base de données. Pour supprimer la base de données, supprimez le fichier de base de données (*MvcMovie.db*). Exécutez ensuite la commande `ef database update` :

```dotnetcli
dotnet ef database update
```

---
<!-- End of VS tabs -->

Exécutez l’application et vérifiez que vous pouvez créer/modifier/afficher des films avec un champ `Rating`. Vous devez ajouter le champ `Rating` aux modèles de vue `Edit`, `Details` et `Delete`.

> [!div class="step-by-step"]
> [Précédent](search.md)
> [Suivant](validation.md)
