---
title: 'Didacticiel : créer une API Web avec ASP.NET Core'
author: rick-anderson
description: Apprendre à créer une API web avec ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 09/29/2019
uid: tutorials/first-web-api
ms.openlocfilehash: 6f2d62600da828261ecfc3a1df688ce914eccf33
ms.sourcegitcommit: a166291c6708f5949c417874108332856b53b6a9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2019
ms.locfileid: "72590014"
---
# <a name="tutorial-create-a-web-api-with-aspnet-core"></a>Didacticiel : créer une API Web avec ASP.NET Core

De [Rick Anderson](https://twitter.com/RickAndMSFT) et [Mike Wasson](https://github.com/mikewasson)

Ce tutoriel décrit les principes fondamentaux liés à la génération d’une API web avec ASP.NET Core.

::: moniker range=">= aspnetcore-3.0"

Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Créer un projet d’API web.
> * Ajouter une classe de modèle et un contexte de base de données.
> * Générer automatiquement des modèles pour un contrôleur avec des méthodes CRUD.
> * Configurer le routage, les chemins d’URL et les valeurs de retour.
> * Appeler l’API web avec Postman

À la fin, vous disposez d’une API web qui peut gérer des tâches stockées dans une base de données.

## <a name="overview"></a>Vue d'ensemble

Ce didacticiel crée l’API suivante :

|API | Description | Corps de la requête | Corps de réponse |
|--- | ---- | ---- | ---- |
|GET /api/TodoItems | Obtenir toutes les tâches | aucune. | Tableau de tâches|
|GET /api/TodoItems/{id} | Obtenir un élément par ID | aucune. | Tâche|
|POST /api/TodoItems | Ajouter un nouvel élément | Tâche | Tâche |
|PUT /api/TodoItems/{id} | Mettre à jour un élément existant &nbsp; | Tâche | aucune. |
|DELETE /api/TodoItems/{id} &nbsp; &nbsp; | Supprimer un élément &nbsp; &nbsp; | aucune. | aucune.|

Le diagramme suivant illustre la conception de l’application.

![Le client est représenté par une zone située à gauche. Il envoie une demande et reçoit une réponse de l’application, représentée par une zone dessinée à droite. Dans la zone de l’application, trois zones représentent le contrôleur, le modèle et la couche d’accès aux données. La requête provient du contrôleur de l’application, et les opérations de lecture/écriture se produisent entre le contrôleur et la couche d’accès aux données. Le modèle est sérialisé et retourné au client dans la réponse.](first-web-api/_static/architecture.png)

## <a name="prerequisites"></a>Configuration requise

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

## <a name="create-a-web-project"></a>Créer un projet web

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dans le menu **Fichier**, sélectionnez **Nouveau** > **Projet**.
* Sélectionnez le modèle **Application web ASP.NET Core** et cliquez sur **Suivant**.
* Nommez le projet *TodoApi* et cliquez sur **Créer**.
* Dans la boîte de dialogue **Créer une application web ASP.NET Core**, vérifiez que **.NET Core** et **ASP.NET Core 3.0** sont sélectionnés. Sélectionnez le modèle **API** et cliquez sur **Créer**.

![Boîte de dialogue de nouveau projet dans VS](first-web-api/_static/vs3.png)

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Ouvrez le [terminal intégré](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Définissez les répertoires (`cd`) sur le dossier destiné à contenir le dossier du projet.
* Exécutez les commandes suivantes :

   ```dotnetcli
   dotnet new webapi -o TodoApi
   cd TodoApi
   dotnet add package Microsoft.EntityFrameworkCore.SqlServer
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   code -r ../TodoApi
   ```

* Quand une boîte de dialogue vous demande si vous souhaitez ajouter les composants nécessaires au projet, sélectionnez **Oui**.

  Les commandes précédentes :

  * Créent un projet API Web et l’ouvrent dans Visual Studio Code.
  * Ajoutent les packages NuGet qui sont requis dans la section suivante.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

* Sélectionnez **Fichier** > **Nouvelle solution**.

  ![macOS - Nouvelle solution](first-web-api-mac/_static/sln.png)

* Sélectionnez **.NET Core** > **Application** > **API** > **Suivant**.

  ![macOS - Boîte de dialogue Nouveau projet](first-web-api-mac/_static/1.png)
  
* Dans la boîte de dialogue **Configurer votre nouvelle API web ASP.NET Core**, sélectionnez *.NET Core 3.0* comme **Framework cible**.

* Entrez *TodoApi* comme **Nom du projet**, puis sélectionnez **Créer**.

  ![boîte de dialogue de configuration](first-web-api-mac/_static/2.png)

[!INCLUDE[](~/includes/mac-terminal-access.md)]

Ouvrez un terminal de commande dans le dossier de projet, puis exécutez les commandes suivantes :

   ```dotnetcli
   dotnet add package Microsoft.EntityFrameworkCore.SqlServer
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   ```

---

### <a name="test-the-api"></a>Tester l’API

Le modèle de projet crée une API `WeatherForecast`. Appelez la méthode `Get` à partir d’un navigateur pour tester l’application.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Appuyez sur Ctrl+F5 pour exécuter l’application. Visual Studio lance un navigateur et accède à `https://localhost:<port>/WeatherForecast`, où `<port>` est un numéro de port choisi de manière aléatoire.

Si une boîte de dialogue apparaît vous demandant si vous devez approuver le certificat IIS Express, sélectionnez **Oui**. Dans la boîte de dialogue **Avertissement de sécurité** qui s’affiche ensuite, sélectionnez **Oui**.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Appuyez sur Ctrl+F5 pour exécuter l’application. Dans un navigateur, accédez à l’URL suivante : [https://localhost:5001/WeatherForecast](https://localhost:5001/WeatherForecast).

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

Sélectionnez **Exécuter** > **Démarrer le débogage** pour lancer l’application. Visual Studio pour Mac lance un navigateur et accède à `https://localhost:<port>`, où `<port>` est un numéro de port choisi de manière aléatoire. Une erreur HTTP 404 (introuvable) est retournée. Ajoutez `/WeatherForecast` à l’URL (définissez-la sur `https://localhost:<port>/WeatherForecast`).

---

Un code JSON similaire au suivant est retourné :

```json
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
```

## <a name="add-a-model-class"></a>Ajouter une classe de modèle

Un *modèle* est un ensemble de classes qui représentent les données gérées par l’application. Le modèle pour cette application est une classe `TodoItem` unique.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet. Sélectionnez **Ajouter** > **Nouveau dossier**. Nommez le dossier *Models*.

* Cliquez avec le bouton droit sur le dossier *Models* et sélectionnez **Ajouter** > **Classe**. Nommez la classe *TodoItem* et sélectionnez sur **Ajouter**.

* Remplacez le code du modèle par le code suivant :

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Ajoutez un dossier nommé *Models*.

* Ajoutez une classe `TodoItem` au dossier *Models* avec le code suivant :

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

* Cliquez avec le bouton droit sur le projet. Sélectionnez **Ajouter** > **Nouveau dossier**. Nommez le dossier *Models*.

  ![nouveau dossier](first-web-api-mac/_static/folder.png)

* Cliquez avec le bouton droit sur le dossier *Models* et sélectionnez **Ajouter** > **Nouveau fichier** > **Général** > **Classe vide**.

* Nommez la classe *TodoItem* et cliquez sur **Nouveau**.

* Remplacez le code du modèle par le code suivant :

---

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Models/TodoItem.cs)]

La propriété `Id` fonctionne comme la clé unique dans une base de données relationnelle.

Vous pouvez placer des classes de modèle n’importe où dans le projet, mais le dossier *Models* est utilisé par convention.

## <a name="add-a-database-context"></a>Ajouter un contexte de base de données

Le *contexte de base de données* est la classe principale qui coordonne les fonctionnalités d’Entity Framework pour un modèle de données. Cette classe est créée en dérivant de la classe `Microsoft.EntityFrameworkCore.DbContext`.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="add-microsoftentityframeworkcoresqlserver"></a>Ajouter Microsoft.EntityFrameworkCore.SqlServer

* Dans le menu **Outils**, sélectionnez **Gestionnaire de package NuGet > Gérer les packages NuGet pour la solution**.
* Sélectionnez l’onglet **Parcourir**, puis entrez **Microsoft.EntityFrameworkCore.SqlServer** dans la zone de recherche.
* Dans le volet gauche, sélectionnez **Microsoft. EntityFrameworkCore. SqlServer** .
* Cochez la case **Projet** dans le volet droit, puis sélectionnez **Installer**.
* Utilisez les instructions précédentes pour ajouter le package NuGet `Microsoft.EntityFrameworkCore.InMemory`.

![NuGet Package Manager](first-web-api/_static/vs3NuGet.png)

## <a name="add-the-todocontext-database-context"></a>Ajouter le contexte de base de données TodoContext

* Cliquez avec le bouton droit sur le dossier *Models* et sélectionnez **Ajouter** > **Classe**. Nommez la classe *TodoContext* et cliquez sur **Ajouter**.

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

* Ajoutez une classe `TodoContext` au dossier *Models*.

---

* Entrez le code suivant :

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context"></a>Inscrire le contexte de base de données

Dans ASP.NET Core, les services tels que le contexte de base de données doivent être inscrits auprès du [conteneur d’injection de dépendances](xref:fundamentals/dependency-injection). Le conteneur fournit le service aux contrôleurs.

Mettez à jour *Startup.cs* avec le code en surbrillance suivant :

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Startup.cs?highlight=7-8,23-24&name=snippet_all)]

Le code précédent :

* Supprime les déclarations `using` inutilisées.
* Ajoute le contexte de base de données au conteneur d’injection de dépendances.
* Spécifie que le contexte de base de données utilise une base de données en mémoire.

## <a name="scaffold-a-controller"></a>Générer automatiquement des modèles pour un contrôleur

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Cliquez avec le bouton droit sur le dossier *Contrôleurs*.
* Sélectionnez **Ajouter** > **Nouvel élément généré automatiquement**.
* Sélectionnez **Contrôleur d’API avec actions, utilisant Entity Framework**, puis **Ajouter**.
* Dans la boîte de dialogue **Contrôleur d’API avec actions, utilisant Entity Framework** :

  * Sélectionnez **TodoItem (TodoApi. Models)** dans la **classe de modèle**.
  * Sélectionnez **TodoContext (TodoApi. Models)** dans la **classe de contexte de données**.
  * Sélectionnez **Ajouter** .

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

Exécutez les commandes suivantes :

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

Les commandes précédentes :

* Ajoutent les packages NuGet nécessaires à la génération de modèles automatique.
* Installent le moteur de génération de modèles automatique (`dotnet-aspnet-codegenerator`).
* Génèrent automatiquement des modèles pour `TodoItemsController`.

---

Le code généré :

* Définit une classe de contrôleur d’API sans méthodes.
* Décore la classe avec l’attribut [[ApiController]](/dotnet/api/microsoft.aspnetcore.mvc.apicontrollerattribute). Cet attribut indique que le contrôleur répond aux requêtes de l’API web. Pour plus d’informations sur les comportements spécifiques que permet l’attribut, consultez <xref:web-api/index>.
* Utilise l’injection de dépendances pour injecter le contexte de base de données (`TodoContext`) dans le contrôleur. Le contexte de base de données est utilisé dans chacune des méthodes la [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) du contrôleur.

## <a name="examine-the-posttodoitem-create-method"></a>Examiner la méthode de création de PostTodoItem

Remplacez l’instruction return dans `PostTodoItem` pour utiliser l’opérateur [nameof](/dotnet/csharp/language-reference/operators/nameof) :

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Create)]

Le code précédent est une méthode HTTP POST, comme indiqué par l’attribut [[HttpPost]](/dotnet/api/microsoft.aspnetcore.mvc.httppostattribute). La méthode obtient la valeur de la tâche dans le corps de la requête HTTP.

La méthode <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> :

* Retourne un code d’état HTTP 201 en cas de réussite. HTTP 201 est la réponse standard d’une méthode HTTP POST qui crée une ressource sur le serveur.
* Ajoute un en-tête [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) à la réponse. L’en-tête `Location` spécifie l’[URI](https://developer.mozilla.org/docs/Glossary/URI) de l’élément d’action qui vient d’être créé. Pour plus d’informations, consultez la section [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
* Fait référence à l’action `GetTodoItem` pour créer l’URI `Location` de l’en-tête. Le mot clé `nameof` C# est utilisé pour éviter de coder en dur le nom de l’action dans l’appel `CreatedAtAction`.

### <a name="install-postman"></a>Installer Postman

Ce tutoriel utilise Postman pour tester l’API web.

* Installez [Postman](https://www.getpostman.com/downloads/).
* Démarrez l’application web.
* Démarrez Postman.
* Désactivez la **vérification du certificat SSL**.
* À partir de **Fichier** > **Paramètres** (onglet **Général**), désactivez **Vérification du certificat SSL**.
    > [!WARNING]
    > Réactivez la vérification du certificat SSL après avoir testé le contrôleur.

<a name="post"></a>

### <a name="test-posttodoitem-with-postman"></a>Tester PostTodoItem avec Postman

* Créez une requête.
* Affectez `POST` à la méthode HTTP.
* Sélectionnez l’onglet **Body** (Corps).
* Sélectionnez la case d’option **raw** (données brutes).
* Définissez le type sur **JSON (application/json)** .
* Dans le corps de la demande, entrez la syntaxe JSON d’une tâche :

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* Sélectionnez **Send** (Envoyer).

  ![Postman avec requête de création](first-web-api/_static/3/create.png)

### <a name="test-the-location-header-uri"></a>Tester l’URI de l’en-tête d’emplacement

* Sélectionnez l’onglet **Headers** (En-têtes) dans le volet **Response** (Réponse).
* Copiez la valeur d’en-tête **Location** (Emplacement) :

  ![Onglet Headers de la console Postman](first-web-api/_static/3/create.png)

* Définissez la méthode sur GET.
* Collez l’URI (par exemple, `https://localhost:5001/api/TodoItems/1`).
* Sélectionnez **Send** (Envoyer).

## <a name="examine-the-get-methods"></a>Examiner les méthodes GET

Ces méthodes implémentent deux points de terminaison GET :

* `GET /api/TodoItems`
* `GET /api/TodoItems/{id}`

Testez l’application en appelant les deux points de terminaison à partir d’un navigateur ou de Postman. Exemple :

* [https://localhost:5001/api/TodoItems](https://localhost:5001/api/TodoItems)
* [https://localhost:5001/api/TodoItems/1](https://localhost:5001/api/TodoItems/1)

Une réponse semblable à la suivante est produite par l’appel à `GetTodoItems` :

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

### <a name="test-get-with-postman"></a>Tester Get avec Postman

* Créez une requête.
* Définissez la méthode HTTP sur **GET**.
* Définissez l’URL de la requête sur `https://localhost:<port>/api/TodoItems`. Par exemple, `https://localhost:5001/api/TodoItems`.
* Définissez l’**affichage à deux volets** dans Postman.
* Sélectionnez **Send** (Envoyer).

Cette application utilise une base de données en mémoire. Si l’application est arrêtée et démarrée, la requête GET précédente ne retourne aucune donnée. Si aucune donnée n’est retournée, publiez ([POST](#post)) les données dans l’application.

## <a name="routing-and-url-paths"></a>Routage et chemins d’URL

L’attribut [`[HttpGet]`](/dotnet/api/microsoft.aspnetcore.mvc.httpgetattribute) désigne une méthode qui répond à une requête HTTP GET. Le chemin d’URL pour chaque méthode est construit comme suit :

* Partez de la chaîne de modèle dans l’attribut `Route` du contrôleur :

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=TodoController&highlight=1)]

* Remplacez `[controller]` par le nom du contrôleur qui, par convention, est le nom de la classe du contrôleur sans le suffixe « Controller ». Pour cet exemple, le nom de la classe du contrôleur étant **TodoItems**Controller, le nom du contrôleur est « TodoItems ». Le [routage](xref:mvc/controllers/routing) d’ASP.NET Core ne respecte pas la casse.
* Si l’attribut `[HttpGet]` a un modèle de route (par exemple, `[HttpGet("products")]`), ajoutez-le au chemin. Cet exemple n’utilise pas de modèle. Pour plus d’informations, consultez [Routage par attributs avec des attributs Http[Verbe]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).

Dans la méthode `GetTodoItem` suivante, `"{id}"` est une variable d’espace réservé pour l’identificateur unique de la tâche. Lorsque `GetTodoItem` est appelé, la valeur de `"{id}"` dans l’URL est fournie à la méthode dans son paramètre `id`.

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values"></a>Valeurs de retour

Le type de retour des méthodes `GetTodoItems` et `GetTodoItem` est [type ActionResult\<T>](xref:web-api/action-return-types#actionresultt-type). ASP.NET Core sérialise automatiquement l’objet en [JSON](https://www.json.org/) et écrit le JSON dans le corps du message de réponse. Le code de réponse pour ce type de retour est 200, en supposant qu’il n’existe pas d’exception non gérée. Les exceptions non gérées sont converties en erreurs 5xx.

Les types de retour `ActionResult` peuvent représenter une large plage de codes d’état HTTP. Par exemple, `GetTodoItem` peut retourner deux valeurs d’état différentes :

* Si aucun élément ne correspond à l’ID demandé, la méthode retourne un code d’erreur 404 [introuvable](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.notfound).
* Sinon, la méthode retourne 200 avec un corps de réponse JSON. Le retour de `item` entraîne une réponse HTTP 200.

## <a name="the-puttodoitem-method"></a>Méthode PutTodoItem

Examinez la méthode `PutTodoItem` :

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Update)]

`PutTodoItem` est similaire à `PostTodoItem`, à la différence près qu’il utilise HTTP PUT. La réponse est [204 (Pas de contenu)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). D’après la spécification HTTP, une requête PUT nécessite que le client envoie toute l’entité mise à jour, et pas seulement les changements. Pour prendre en charge les mises à jour partielles, utilisez [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).

Si vous obtenez une erreur en appelant `PutTodoItem`, appelez `GET` pour vérifier que la base de données contient un élément.

### <a name="test-the-puttodoitem-method"></a>Tester la méthode PutTodoItem

Cet exemple utilise une base de données en mémoire qui doit être initialisée chaque fois que l’application est démarrée. La base de données doit contenir un élément avant que vous ne passiez un appel PUT. Appelez GET pour vérifier qu’un élément existe dans la base de données avant d’effectuer un appel PUT.

Mettez à jour la tâche dont l’ID = 1 et nommez-la « feed fish » :

```json
  {
    "ID":1,
    "name":"feed fish",
    "isComplete":true
  }
```

L’image suivante montre la mise à jour Postman :

![Console Postman montrant la réponse 204 (Pas de contenu)](first-web-api/_static/3/pmcput.png)

## <a name="the-deletetodoitem-method"></a>Méthode DeleteTodoItem

Examinez la méthode `DeleteTodoItem` :

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Delete)]

La réponse `DeleteTodoItem` est [204 (Pas de contenu)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).

### <a name="test-the-deletetodoitem-method"></a>Tester la méthode DeleteTodoItem

Utilisez Postman pour supprimer une tâche :

* Définissez la méthode sur `DELETE`.
* Définissez l’URI de l’objet à supprimer (par exemple `https://localhost:5001/api/TodoItems/1`).
* Sélectionnez **Send** (Envoyer).

## <a name="call-the-web-api-with-javascript"></a>Appelez l’API web avec JavaScript

Consultez [Didacticiel : appeler une API web ASP.net core avec JavaScript](xref:tutorials/web-api-javascript).

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Créer un projet d’API web.
> * Ajouter une classe de modèle et un contexte de base de données.
> * Ajouter un contrôleur
> * Ajouter les méthodes CRUD
> * Configurer le routage et les chemins d’URL
> * Spécifier des valeurs de retour
> * Appeler l’API web avec Postman
> * Appeler l’API web avec JavaScript.

À la fin, vous disposez d’une API web qui peut gérer des tâches stockées dans une base de données relationnelle.

## <a name="overview"></a>Vue d'ensemble

Ce didacticiel crée l’API suivante :

|API | Description | Corps de la requête | Corps de réponse |
|--- | ---- | ---- | ---- |
|GET /api/TodoItems | Obtenir toutes les tâches | aucune. | Tableau de tâches|
|GET /api/TodoItems/{id} | Obtenir un élément par ID | aucune. | Tâche|
|POST /api/TodoItems | Ajouter un nouvel élément | Tâche | Tâche |
|PUT /api/TodoItems/{id} | Mettre à jour un élément existant &nbsp; | Tâche | aucune. |
|DELETE /api/TodoItems/{id} &nbsp; &nbsp; | Supprimer un élément &nbsp; &nbsp; | aucune. | aucune.|

Le diagramme suivant illustre la conception de l’application.

![Le client est représenté par une zone située à gauche. Il envoie une demande et reçoit une réponse de l’application, représentée par une zone dessinée à droite. Dans la zone de l’application, trois zones représentent le contrôleur, le modèle et la couche d’accès aux données. La requête provient du contrôleur de l’application, et les opérations de lecture/écriture se produisent entre le contrôleur et la couche d’accès aux données. Le modèle est sérialisé et retourné au client dans la réponse.](first-web-api/_static/architecture.png)

## <a name="prerequisites"></a>Configuration requise

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

## <a name="create-a-web-project"></a>Créer un projet web

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dans le menu **Fichier**, sélectionnez **Nouveau** > **Projet**.
* Sélectionnez le modèle **Application web ASP.NET Core** et cliquez sur **Suivant**.
* Nommez le projet *TodoApi* et cliquez sur **Créer**.
* Dans la boîte de dialogue **Créer une application web ASP.NET Core**, vérifiez que **.NET Core** et **ASP.NET Core 2.2** sont sélectionnés. Sélectionnez le modèle **API** et cliquez sur **Créer**. Ne sélectionnez **pas** **Activer la prise en charge de Docker**.

![Boîte de dialogue de nouveau projet dans VS](first-web-api/_static/vs.png)

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Ouvrez le [terminal intégré](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Définissez les répertoires (`cd`) sur le dossier destiné à contenir le dossier du projet.
* Exécutez les commandes suivantes :

   ```dotnetcli
   dotnet new webapi -o TodoApi
   code -r TodoApi
   ```

  Ces commandes créent un projet d’API web et ouvrent une nouvelle instance de Visual Studio Code dans le nouveau dossier de projet.

* Quand une boîte de dialogue vous demande si vous souhaitez ajouter les composants nécessaires au projet, sélectionnez **Oui**.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

* Sélectionnez **Fichier** > **Nouvelle solution**.

  ![macOS - Nouvelle solution](first-web-api-mac/_static/sln.png)

* Sélectionnez **.NET Core** > **Application** > **API** > **Suivant**.

  ![macOS - Boîte de dialogue Nouveau projet](first-web-api-mac/_static/1.png)
  
* Dans la boîte de dialogue **Configurer votre nouvelle API web ASP.NET Core**, acceptez la valeur par défaut * *.NET Core 2.2* pour **Framework cible**.

* Entrez *TodoApi* comme **Nom du projet**, puis sélectionnez **Créer**.

  ![boîte de dialogue de configuration](first-web-api-mac/_static/2.png)

---

### <a name="test-the-api"></a>Tester l’API

Le modèle de projet crée une API `values`. Appelez la méthode `Get` à partir d’un navigateur pour tester l’application.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Appuyez sur Ctrl+F5 pour exécuter l’application. Visual Studio lance un navigateur et accède à `https://localhost:<port>/api/values`, où `<port>` est un numéro de port choisi de manière aléatoire.

Si une boîte de dialogue apparaît vous demandant si vous devez approuver le certificat IIS Express, sélectionnez **Oui**. Dans la boîte de dialogue **Avertissement de sécurité** qui s’affiche ensuite, sélectionnez **Oui**.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Appuyez sur Ctrl+F5 pour exécuter l’application. Dans un navigateur, accédez à l’URL suivante : [https://localhost:5001/api/values](https://localhost:5001/api/values).

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

Sélectionnez **Exécuter** > **Démarrer le débogage** pour lancer l’application. Visual Studio pour Mac lance un navigateur et accède à `https://localhost:<port>`, où `<port>` est un numéro de port choisi de manière aléatoire. Une erreur HTTP 404 (introuvable) est retournée. Ajoutez `/api/values` à l’URL (définissez-la sur `https://localhost:<port>/api/values`).

---

Le code JSON suivant est retourné :

```json
["value1","value2"]
```

## <a name="add-a-model-class"></a>Ajouter une classe de modèle

Un *modèle* est un ensemble de classes qui représentent les données gérées par l’application. Le modèle pour cette application est une classe `TodoItem` unique.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet. Sélectionnez **Ajouter** > **Nouveau dossier**. Nommez le dossier *Models*.

* Cliquez avec le bouton droit sur le dossier *Models* et sélectionnez **Ajouter** > **Classe**. Nommez la classe *TodoItem* et sélectionnez sur **Ajouter**.

* Remplacez le code du modèle par le code suivant :

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Ajoutez un dossier nommé *Models*.

* Ajoutez une classe `TodoItem` au dossier *Models* avec le code suivant :

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

* Cliquez avec le bouton droit sur le projet. Sélectionnez **Ajouter** > **Nouveau dossier**. Nommez le dossier *Models*.

  ![nouveau dossier](first-web-api-mac/_static/folder.png)

* Cliquez avec le bouton droit sur le dossier *Models* et sélectionnez **Ajouter** > **Nouveau fichier** > **Général** > **Classe vide**.

* Nommez la classe *TodoItem* et cliquez sur **Nouveau**.

* Remplacez le code du modèle par le code suivant :

---

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Models/TodoItem.cs)]

La propriété `Id` fonctionne comme la clé unique dans une base de données relationnelle.

Vous pouvez placer des classes de modèle n’importe où dans le projet, mais le dossier *Models* est utilisé par convention.

## <a name="add-a-database-context"></a>Ajouter un contexte de base de données

Le *contexte de base de données* est la classe principale qui coordonne les fonctionnalités d’Entity Framework pour un modèle de données. Cette classe est créée en dérivant de la classe `Microsoft.EntityFrameworkCore.DbContext`.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Cliquez avec le bouton droit sur le dossier *Models* et sélectionnez **Ajouter** > **Classe**. Nommez la classe *TodoContext* et cliquez sur **Ajouter**.

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

* Ajoutez une classe `TodoContext` au dossier *Models*.

---

* Remplacez le code du modèle par le code suivant :

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context"></a>Inscrire le contexte de base de données

Dans ASP.NET Core, les services tels que le contexte de base de données doivent être inscrits auprès du [conteneur d’injection de dépendances](xref:fundamentals/dependency-injection). Le conteneur fournit le service aux contrôleurs.

Mettez à jour *Startup.cs* avec le code en surbrillance suivant :

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Startup1.cs?highlight=5,8,25-26&name=snippet_all)]

Le code précédent :

* Supprime les déclarations `using` inutilisées.
* Ajoute le contexte de base de données au conteneur d’injection de dépendances.
* Spécifie que le contexte de base de données utilise une base de données en mémoire.

## <a name="add-a-controller"></a>Ajouter un contrôleur

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Cliquez avec le bouton droit sur le dossier *Contrôleurs*.
* Sélectionnez **Ajouter** > **Nouvel élément**.
* Dans la boîte de dialogue **Ajouter un nouvel élément**, sélectionnez le modèle **Classe de contrôleur d’API**.
* Nommez la classe *TodoController* et sélectionnez **Ajouter**.

  ![Boîte de dialogue Ajouter un nouvel élément avec contrôleur dans la zone de recherche et le contrôleur des API web sélectionné](first-web-api/_static/new_controller.png)

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

* Dans le dossier *Controllers*, créez une classe nommée `TodoController`.

---

* Remplacez le code du modèle par le code suivant :

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController2.cs?name=snippet_todo1)]

Le code précédent :

* Définit une classe de contrôleur d’API sans méthodes.
* Décore la classe avec l’attribut [[ApiController]](/dotnet/api/microsoft.aspnetcore.mvc.apicontrollerattribute). Cet attribut indique que le contrôleur répond aux requêtes de l’API web. Pour plus d’informations sur les comportements spécifiques que permet l’attribut, consultez <xref:web-api/index>.
* Utilise l’injection de dépendances pour injecter le contexte de base de données (`TodoContext`) dans le contrôleur. Le contexte de base de données est utilisé dans chacune des méthodes la [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) du contrôleur.
* Ajoute un élément nommé `Item1` à la base de données si celle-ci est vide. Ce code se trouvant dans le constructeur, il s’exécute chaque fois qu’une nouvelle requête HTTP existe. Si vous supprimez tous les éléments, le constructeur recrée `Item1` au prochain appel d’une méthode d’API. Ainsi, il peut vous sembler à tort que la suppression n’a pas fonctionné.

## <a name="add-get-methods"></a>Ajouter des méthodes Get

Pour fournir une API qui récupère les tâches, ajoutez les méthodes suivantes à la classe `TodoController` :

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetAll)]

Ces méthodes implémentent deux points de terminaison GET :

* `GET /api/todo`
* `GET /api/todo/{id}`

Arrêtez l’application si elle est toujours en cours d’exécution. Ensuite, réexécutez-la pour inclure les dernières modifications.

Testez l’application en appelant les deux points de terminaison à partir d’un navigateur. Exemple :

* `https://localhost:<port>/api/todo`
* `https://localhost:<port>/api/todo/1`

La réponse HTTP suivante est générée par l’appel à `GetTodoItems` :

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

## <a name="routing-and-url-paths"></a>Routage et chemins d’URL

L’attribut [`[HttpGet]`](/dotnet/api/microsoft.aspnetcore.mvc.httpgetattribute) désigne une méthode qui répond à une requête HTTP GET. Le chemin d’URL pour chaque méthode est construit comme suit :

* Partez de la chaîne de modèle dans l’attribut `Route` du contrôleur :

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=TodoController&highlight=3)]

* Remplacez `[controller]` par le nom du contrôleur qui, par convention, est le nom de la classe du contrôleur sans le suffixe « Controller ». Pour cet exemple, le nom de classe du contrôleur étant **Todo**Controller, le nom du contrôleur est « todo ». Le [routage](xref:mvc/controllers/routing) d’ASP.NET Core ne respecte pas la casse.
* Si l’attribut `[HttpGet]` a un modèle de route (par exemple, `[HttpGet("products")]`), ajoutez-le au chemin. Cet exemple n’utilise pas de modèle. Pour plus d’informations, consultez [Routage par attributs avec des attributs Http[Verbe]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).

Dans la méthode `GetTodoItem` suivante, `"{id}"` est une variable d’espace réservé pour l’identificateur unique de la tâche. Quand `GetTodoItem` est appelée, la valeur de `"{id}"` dans l’URL est fournie à la méthode dans son paramètre `id`.

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values"></a>Valeurs de retour

Le type de retour des méthodes `GetTodoItems` et `GetTodoItem` est [type ActionResult\<T>](xref:web-api/action-return-types#actionresultt-type). ASP.NET Core sérialise automatiquement l’objet en [JSON](https://www.json.org/) et écrit le JSON dans le corps du message de réponse. Le code de réponse pour ce type de retour est 200, en supposant qu’il n’existe pas d’exception non gérée. Les exceptions non gérées sont converties en erreurs 5xx.

Les types de retour `ActionResult` peuvent représenter une large plage de codes d’état HTTP. Par exemple, `GetTodoItem` peut retourner deux valeurs d’état différentes :

* Si aucun élément ne correspond à l’ID demandé, la méthode retourne un code d’erreur 404 [introuvable](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.notfound).
* Sinon, la méthode retourne 200 avec un corps de réponse JSON. Le retour de `item` entraîne une réponse HTTP 200.

## <a name="test-the-gettodoitems-method"></a>Tester la méthode GetTodoItems

Ce tutoriel utilise Postman pour tester l’API web.

* Installez le [poste de publication](https://www.getpostman.com/downloads/).
* Démarrez l’application web.
* Démarrez Postman.
* Désactivez la **vérification du certificat SSL**.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* À partir de **Fichier** > **Paramètres** (onglet **Général**), désactivez **Vérification du certificat SSL**.

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio pour Mac](#tab/visual-studio-code+visual-studio-mac)

* À partir de **Postman** > **Préférences** (onglet **Général**), désactivez **Vérification du certificat SSL**. Vous pouvez également sélectionner la clé et sélectionner **Paramètres**, puis désactiver la vérification du certificat SSL.

---
  
> [!WARNING]
> Réactivez la vérification du certificat SSL après avoir testé le contrôleur.

* Créez une requête.
  * Définissez la méthode HTTP sur **GET**.
  * Définissez l’URL de la requête sur `https://localhost:<port>/api/todo`. Par exemple, `https://localhost:5001/api/todo`.
* Définissez l’**affichage à deux volets** dans Postman.
* Sélectionnez **Send** (Envoyer).

![Postman avec requête Get](first-web-api/_static/2pv.png)

## <a name="add-a-create-method"></a>Ajouter une méthode Create

Ajoutez la méthode `PostTodoItem` suivante à *Controllers/TodoController.cs* : 

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Create)]

Le code précédent est une méthode HTTP POST, comme indiqué par l’attribut [[HttpPost]](/dotnet/api/microsoft.aspnetcore.mvc.httppostattribute). La méthode obtient la valeur de la tâche dans le corps de la requête HTTP.

La méthode `CreatedAtAction` :

* Retourne un code d’état HTTP 201 en cas de réussite. HTTP 201 est la réponse standard d’une méthode HTTP POST qui crée une ressource sur le serveur.
* Ajoute un en-tête `Location` à la réponse. L’en-tête `Location` spécifie l’URI de l’élément d’action qui vient d’être créé. Pour plus d’informations, consultez la section [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
* Fait référence à l’action `GetTodoItem` pour créer l’URI `Location` de l’en-tête. Le mot clé `nameof` C# est utilisé pour éviter de coder en dur le nom de l’action dans l’appel `CreatedAtAction`.

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetByID&highlight=1-2)]

### <a name="test-the-posttodoitem-method"></a>Tester la méthode PostTodoItem

* Générez le projet.
* Dans Postman, définissez la méthode HTTP sur `POST`.
* Sélectionnez l’onglet **Body** (Corps).
* Sélectionnez la case d’option **raw** (données brutes).
* Définissez le type sur **JSON (application/json)** .
* Dans le corps de la demande, entrez la syntaxe JSON d’une tâche :

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* Sélectionnez **Send** (Envoyer).

  ![Postman avec requête de création](first-web-api/_static/create.png)

  Si vous obtenez une erreur 405 Méthode non autorisée, il est probable que le projet n’ait pas été compilé après l’ajout de la méthode `PostTodoItem`.

### <a name="test-the-location-header-uri"></a>Tester l’URI de l’en-tête d’emplacement

* Sélectionnez l’onglet **Headers** (En-têtes) dans le volet **Response** (Réponse).
* Copiez la valeur d’en-tête **Location** (Emplacement) :

  ![Onglet Headers de la console Postman](first-web-api/_static/pmc2.png)

* Définissez la méthode sur GET.
* Collez l’URI (par exemple, `https://localhost:5001/api/Todo/2`).
* Sélectionnez **Send** (Envoyer).

## <a name="add-a-puttodoitem-method"></a>Ajouter une méthode PutTodoItem

Ajoutez la méthode `PutTodoItem` suivante :

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Update)]

`PutTodoItem` est similaire à `PostTodoItem`, à la différence près qu’il utilise HTTP PUT. La réponse est [204 (Pas de contenu)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). D’après la spécification HTTP, une requête PUT nécessite que le client envoie toute l’entité mise à jour, et pas seulement les changements. Pour prendre en charge les mises à jour partielles, utilisez [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).

Si vous obtenez une erreur en appelant `PutTodoItem`, appelez `GET` pour vérifier que la base de données contient un élément.

### <a name="test-the-puttodoitem-method"></a>Tester la méthode PutTodoItem

Cet exemple utilise une base de données en mémoire qui doit être initialisée chaque fois que l’application est démarrée. La base de données doit contenir un élément avant que vous ne passiez un appel PUT. Appelez GET pour vérifier qu’un élément existe dans la base de données avant d’effectuer un appel PUT.

Mettez à jour la tâche dont l’id est 1 en définissant son nom sur « feed fish » :

```json
  {
    "ID":1,
    "name":"feed fish",
    "isComplete":true
  }
```

L’image suivante montre la mise à jour Postman :

![Console Postman montrant la réponse 204 (Pas de contenu)](first-web-api/_static/pmcput.png)

## <a name="add-a-deletetodoitem-method"></a>Ajouter une méthode DeleteTodoItem

Ajoutez la méthode `DeleteTodoItem` suivante :

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Delete)]

La réponse `DeleteTodoItem` est [204 (Pas de contenu)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).

### <a name="test-the-deletetodoitem-method"></a>Tester la méthode DeleteTodoItem

Utilisez Postman pour supprimer une tâche :

* Définissez la méthode sur `DELETE`.
* Définissez l’URI de l’objet à supprimer (par exemple `https://localhost:5001/api/todo/1`).
* Sélectionnez **Send** (Envoyer).

L’exemple d’application vous permet de supprimer tous les éléments. Toutefois, quand le dernier élément est supprimé, un autre est créé par le constructeur de classe de modèle au prochain appel de l’API.

## <a name="call-the-web-api-with-javascript"></a>Appelez l’API web avec JavaScript

Dans cette section, une page HTML qui utilise JavaScript pour appeler l’API web est ajoutée. jQuery lance la demande. JavaScript met à jour la page avec les détails de la réponse de l’API Web.

Configurez l’application pour [traiter les fichiers statiques](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) et [activer le mappage de fichier par défaut](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) en mettant à jour *Startup.cs* avec le code en surbrillance suivant :

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Startup.cs?highlight=14-15&name=snippet_configure)]

Créez un dossier *wwwroot* dans le répertoire du projet.

Ajoutez un fichier HTML nommé *index.html* au répertoire *wwwroot*. Remplacez son contenu par le balisage suivant :

[!code-html[](first-web-api/samples/2.2/TodoApi/wwwroot/index.html)]

Ajoutez un fichier JavaScript nommé *site.js* au répertoire *wwwroot*. Remplacez son contenu par le code suivant :

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_SiteJs)]

Vous devrez peut-être changer les paramètres de lancement du projet ASP.NET Core pour tester la page HTML localement :

* Ouvrez *Properties\launchSettings.json*.
* Supprimez la propriété `launchUrl` pour forcer l’utilisation du fichier *index.html* (fichier par défaut du projet) à l’ouverture de l’application.

Cet exemple appelle toutes les méthodes CRUD de l’API web. Les explications suivantes traitent des appels à l’API.

### <a name="get-a-list-of-to-do-items"></a>Obtenir une liste de tâches

jQuery envoie une requête HTTP obtenir à l’API Web, qui retourne JSON représentant un tableau d’éléments de tâche. La fonction de rappel `success` est appelée si la requête réussit. Dans le rappel, le DOM est mis à jour avec les informations des tâches.

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_GetData)]

### <a name="add-a-to-do-item"></a>Ajouter une tâche

jQuery envoie une requête HTTP POSTÉRIEURe avec l’élément de tâche dans le corps de la demande. Les options `accepts` et `contentType` sont définies avec la valeur `application/json` pour spécifier le type de média qui est reçu et envoyé. La tâche est convertie au format JSON à l’aide de [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify). Quand l’API retourne un code d’état de réussite, la fonction `getData` est appelée pour mettre à jour la table HTML.

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AddItem)]

### <a name="update-a-to-do-item"></a>Mettre à jour une tâche

La mise à jour d’une tâche est similaire à l’ajout d’une tâche. L’identificateur unique de la tâche est ajouté à l’`url` et le `type` est `PUT`.

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AjaxPut)]

### <a name="delete-a-to-do-item"></a>Supprimer une tâche

Pour supprimer une tâche, vous devez définir le `type` sur l’appel AJAX avec la valeur `DELETE` et spécifier l’identificateur unique de l’élément dans l’URL.

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AjaxDelete)]

::: moniker-end

<a name="auth"></a>

## <a name="add-authentication-support-to-a-web-api"></a>Ajouter la prise en charge de l’authentification à une API Web

Consultez le didacticiel [IdentityServer4](https://identityserver4.readthedocs.io/en/latest/quickstarts/0_overview.html) .

## <a name="additional-resources"></a>Ressources supplémentaires

[Affichez ou téléchargez l’exemple de code de ce tutoriel](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/first-web-api/samples). Consultez [Guide pratique pour télécharger](xref:index#how-to-download-a-sample).

Pour plus d'informations, reportez-vous aux ressources suivantes :

* <xref:web-api/index>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:data/ef-rp/intro>
* <xref:mvc/controllers/routing>
* <xref:web-api/action-return-types>
* <xref:host-and-deploy/azure-apps/index>
* <xref:host-and-deploy/index>
* [Version YouTube de ce tutoriel](https://www.youtube.com/watch?v=TTkhEyGBfAk)
