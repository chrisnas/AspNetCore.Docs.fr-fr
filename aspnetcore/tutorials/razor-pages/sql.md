---
title: Utiliser une base de données et ASP.NET Core
author: rick-anderson
description: Explique l’utilisation d’une base de données et d’ASP.NET Core.
ms.author: riande
ms.date: 7/22/2019
uid: tutorials/razor-pages/sql
ms.openlocfilehash: b5acb573f8fa39e5300ecdb359113d8697d78934
ms.sourcegitcommit: 07d98ada57f2a5f6d809d44bdad7a15013109549
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2019
ms.locfileid: "72334228"
---
# <a name="work-with-a-database-and-aspnet-core"></a>Utiliser une base de données et ASP.NET Core

Par [Rick Anderson](https://twitter.com/RickAndMSFT) et [Joe Audette](https://twitter.com/joeaudette)

::: moniker range=">= aspnetcore-3.0"

[!INCLUDE[](~/includes/rp/download.md)]

L’objet `RazorPagesMovieContext` gère la tâche de connexion à la base de données et de mappage d’objets `Movie` à des enregistrements de la base de données. Le contexte de base de données est inscrit auprès du conteneur [Injection de dépendances](xref:fundamentals/dependency-injection) dans la méthode `ConfigureServices` de *Startup.cs* :

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=15-18)]

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

---

Le système de [configuration](xref:fundamentals/configuration/index) d’ASP.NET Core lit `ConnectionString`. Pour un développement local, il obtient la chaîne de connexion à partir du fichier *appsettings.json*.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

La valeur du nom de la base de données (`Database={Database name}`) est différent pour votre code généré. La valeur du nom est arbitraire.

[!code-json[](razor-pages-start/sample/RazorPagesMovie30/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/appsettings_SQLite.json?highlight=8-10)]

---

Quand l’application est déployée sur un serveur de test ou de production, une variable d’environnement peut être utilisée pour définir la chaîne de connexion à un serveur de base de données réel. Pour plus d’informations, consultez [Configuration](xref:fundamentals/configuration/index).

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

LocalDB est une version allégée du moteur de base de données SQL Server Express, qui est ciblée pour le développement de programmes. LocalDB démarre à la demande et s’exécute en mode utilisateur, ce qui n’implique aucune configuration complexe. Par défaut, la base de données LocalDB crée des fichiers `*.mdf` dans le répertoire `C:\Users\<user>\`.

<a name="ssox"></a>
* Dans le menu **Affichage**, ouvrez **l’Explorateur d’objets SQL Server** (SSOX).

  ![Menu View](sql/_static/ssox.png)

* Cliquez avec le bouton droit sur la table `Movie` et sélectionnez **Concepteur de vues** :

  ![Les menus contextuels s’ouvrent dans la table Movie](sql/_static/design.png)

  ![Les tables Movie s’ouvrent dans le concepteur](sql/_static/dv.png)

Notez l’icône de clé en regard de `ID`. Par défaut, EF crée une propriété nommée `ID` pour la clé primaire.

* Cliquez avec le bouton droit sur la table `Movie` et sélectionnez **Afficher les données** :

  ![Table Movie ouverte, affichant des données de table](sql/_static/vd22.png)

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

---

## <a name="seed-the-database"></a>Amorcer la base de données

Créez une classe nommée `SeedData` dans le dossier *Modèles* avec le code suivant :

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedData.cs?name=snippet_1)]

Si la base de données contient des films, l’initialiseur de valeur initiale retourne une valeur et aucun film n’est ajouté.

```csharp
if (context.Movie.Any())
{
    return;   // DB has been seeded.
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>Ajouter l’initialiseur de valeur initiale

Dans *Program.cs*, modifiez la méthode `Main` pour effectuer les opérations suivantes :

* Obtenir une instance de contexte de base de données à partir du conteneur d’injection de dépendances.
* Appeler la méthode seed et la passer au contexte.
* Supprimer le contexte une fois la méthode seed terminée.

Le code suivant montre le fichier *Program.cs* mis à jour.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Program.cs)]

L’exception suivante se produit lorsque `Update-Database` n’a pas été exécutée :

> `SqlException: Cannot open database "RazorPagesMovieContext-" requested by the login. The login failed.`
> `Login failed for user 'user name'.`

### <a name="test-the-app"></a>Tester l’application

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Supprimez tous les enregistrements de la base de données. Pour ce faire, utilisez les liens de suppression disponibles dans le navigateur ou à partir de [SSOX](xref:tutorials/razor-pages/new-field#ssox)
* Forcez l’application à s’initialiser (appelez les méthodes de la classe `Startup`) pour que la méthode seed s’exécute. Pour forcer l’initialisation, IIS Express doit être arrêté et redémarré. Pour cela, adoptez l’une des approches suivantes :

  * Cliquez avec le bouton droit sur l’icône de barre d’état système IIS Express dans la zone de notification, puis appuyez sur **Quitter** ou sur **Arrêter le site** :

    ![Icône de la barre d’état système IIS Express](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![Menu contextuel](sql/_static/stopIIS.png)

    * Si vous exécutiez Visual Studio en mode de non-débogage, appuyez sur F5 pour l’exécuter en mode de débogage.
    * Si vous exécutiez Visual Studio en mode de débogage, arrêtez le débogueur et appuyez sur F5.

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

Supprimez tous les enregistrements dans la base de données (pour que la méthode seed s’exécute). Arrêtez et démarrez l’application pour amorcer la base de données.

L’application affiche les données de départ.

---

Le tutoriel suivant sert à améliorer la présentation des données.

## <a name="additional-resources"></a>Ressources supplémentaires

> [!div class="step-by-step"]
> [Précédent : Pages Razor obtenues par génération de modèles automatiques](xref:tutorials/razor-pages/page)
> [Suivant : Mises à jour des pages](xref:tutorials/razor-pages/da1)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!INCLUDE[](~/includes/rp/download.md)]

L’objet `RazorPagesMovieContext` gère la tâche de connexion à la base de données et de mappage d’objets `Movie` à des enregistrements de la base de données. Le contexte de base de données est inscrit auprès du conteneur [Injection de dépendances](xref:fundamentals/dependency-injection) dans la méthode `ConfigureServices` de *Startup.cs* :

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-18)]

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

---

Pour plus d’informations sur les méthodes utilisées dans `ConfigureServices`, consultez :

* [Prise en charge du règlement général sur la protection des données (RGPD) de l’Union Européenne dans ASP.NET Core](xref:security/gdpr) pour `CookiePolicyOptions`.
* [SetCompatibilityVersion](xref:mvc/compatibility-version)

Le système de [configuration](xref:fundamentals/configuration/index) d’ASP.NET Core lit `ConnectionString`. Pour un développement local, il obtient la chaîne de connexion à partir du fichier *appsettings.json*.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

La valeur du nom de la base de données (`Database={Database name}`) est différent pour votre code généré. La valeur du nom est arbitraire.

[!code-json[](razor-pages-start/sample/RazorPagesMovie22/appsettings.json)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/appsettings_SQLite.json?highlight=8-10)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/appsettings_SQLite.json?highlight=8-10)]

---

Quand l’application est déployée sur un serveur de test ou de production, une variable d’environnement peut être utilisée pour définir la chaîne de connexion à un serveur de base de données réel. Pour plus d’informations, consultez [Configuration](xref:fundamentals/configuration/index).

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

LocalDB est une version allégée du moteur de base de données SQL Server Express, qui est ciblée pour le développement de programmes. LocalDB démarre à la demande et s’exécute en mode utilisateur, ce qui n’implique aucune configuration complexe. Par défaut, la base de données LocalDB crée des fichiers `*.mdf` dans le répertoire `C:/Users/<user/>`.

<a name="ssox"></a>
* Dans le menu **Affichage**, ouvrez **l’Explorateur d’objets SQL Server** (SSOX).

  ![Menu View](sql/_static/ssox.png)

* Cliquez avec le bouton droit sur la table `Movie` et sélectionnez **Concepteur de vues** :

  ![Menu contextuel ouvert sur la table Movie](sql/_static/design.png)

  ![Table Movie ouverte dans le Concepteur](sql/_static/dv.png)

Notez l’icône de clé en regard de `ID`. Par défaut, EF crée une propriété nommée `ID` pour la clé primaire.

* Cliquez avec le bouton droit sur la table `Movie` et sélectionnez **Afficher les données** :

  ![Table Movie ouverte, affichant des données de table](sql/_static/vd22.png)

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

---

## <a name="seed-the-database"></a>Amorcer la base de données

Créez une classe nommée `SeedData` dans le dossier *Modèles* avec le code suivant :

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedData.cs?name=snippet_1)]

Si la base de données contient des films, l’initialiseur de valeur initiale retourne une valeur et aucun film n’est ajouté.

```csharp
if (context.Movie.Any())
{
    return;   // DB has been seeded.
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>Ajouter l’initialiseur de valeur initiale

Dans *Program.cs*, modifiez la méthode `Main` pour effectuer les opérations suivantes :

* Obtenir une instance de contexte de base de données à partir du conteneur d’injection de dépendances.
* Appeler la méthode seed et la passer au contexte.
* Supprimer le contexte une fois la méthode seed terminée.

Le code suivant montre le fichier *Program.cs* mis à jour.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Program.cs)]

Une application de production n’appelle pas `Database.Migrate`. Il est ajouté au code précédent afin d’éviter l’exception suivante quand `Update-Database` n’a pas été exécutée :

SqlException: impossible d’ouvrir la base de données 'RazorPagesMovieContext-21' demandée par la connexion. La connexion a échoué.
Échec de la connexion de l’utilisateur 'nom utilisateur'.

### <a name="test-the-app"></a>Tester l’application

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Supprimez tous les enregistrements de la base de données. Pour ce faire, utilisez les liens de suppression disponibles dans le navigateur ou à partir de [SSOX](xref:tutorials/razor-pages/new-field#ssox)
* Forcez l’application à s’initialiser (appelez les méthodes de la classe `Startup`) pour que la méthode seed s’exécute. Pour forcer l’initialisation, IIS Express doit être arrêté et redémarré. Pour cela, adoptez l’une des approches suivantes :

  * Cliquez avec le bouton droit sur l’icône de barre d’état système IIS Express dans la zone de notification, puis appuyez sur **Quitter** ou sur **Arrêter le site** :

    ![Icône de la barre d’état système IIS Express](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![Menu contextuel](sql/_static/stopIIS.png)

    * Si vous exécutiez Visual Studio en mode de non-débogage, appuyez sur F5 pour l’exécuter en mode de débogage.
    * Si vous exécutiez Visual Studio en mode de débogage, arrêtez le débogueur et appuyez sur F5.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Supprimez tous les enregistrements dans la base de données (pour que la méthode seed s’exécute). Arrêtez et démarrez l’application pour amorcer la base de données.

L’application affiche les données de départ.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

Supprimez tous les enregistrements dans la base de données (pour que la méthode seed s’exécute). Arrêtez et démarrez l’application pour amorcer la base de données.

L’application affiche les données de départ.

---

L’application affiche les données de départ :

![Application Movie ouverte dans Chrome, affichant les données relatives aux films](sql/_static/m55.png)

Le didacticiel suivant nettoie la présentation des données.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Version YouTube de ce tutoriel](https://youtu.be/A_5ff11sDHY)

> [!div class="step-by-step"]
> [Précédent : Pages Razor obtenues par génération de modèles automatiques](xref:tutorials/razor-pages/page)
> [Suivant : Mises à jour des pages](xref:tutorials/razor-pages/da1)

::: moniker-end
