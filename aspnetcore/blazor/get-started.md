---
title: Prise en main d’ASP.NET Core éblouissante
author: guardrex
description: Commencez avec éblouissant en créant une application éblouissant avec les outils de votre choix.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: blazor/get-started
ms.openlocfilehash: fc368be5eb2e5d8f7c80071dc86a02ae986a685f
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391055"
---
# <a name="get-started-with-aspnet-core-blazor"></a>Prise en main d’ASP.NET Core éblouissante

Par [Daniel Roth](https://github.com/danroth27) et [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Prise en main de éblouissant :

::: moniker range=">= aspnetcore-3.1"

1. Installez le [Kit de développement logiciel (SDK) .net Core 3,1 preview](https://dotnet.microsoft.com/download/dotnet-core/3.1).

1. Installez le modèle de [Webassembly éblouissant](xref:blazor/hosting-models#blazor-webassembly) en exécutant la commande suivante dans une interface de commande. Le package [Microsoft. AspNetCore. éblouissant. Templates](https://www.nuget.org/packages/Microsoft.AspNetCore.Blazor.Templates/) a une préversion alors que l’assembly éblouissant est en version préliminaire.

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview1.19508.20
   ```

1. Suivez les instructions de votre choix d’outils :

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1 \. Installez [Visual Studio 16,4 Preview 2 ou version ultérieure](https://visualstudio.microsoft.com/vs/preview/) avec la charge de travail **développement Web et ASP.net** .

   2 \. Créer un nouveau projet.

   3 \. Sélectionnez l' **application éblouissant**. Sélectionnez **Suivant**.

   4 \. Indiquez un nom de projet dans le champ **Nom du projet**, ou acceptez le nom de projet par défaut. Confirmez que l’entrée d' **emplacement** est correcte ou indiquez un emplacement pour le projet. Sélectionnez **Créer**.

   5 \. Pour une expérience de webassembly éblouissant, choisissez le modèle **application éblouissant Webassembly** . Pour une expérience de serveur éblouissant, choisissez le modèle **application de serveur éblouissant** . Sélectionnez **Créer**. Pour plus d’informations sur les deux modèles d’hébergement éblouissants, le *serveur éblouissant* et le *webassembly éblouissant*, consultez <xref:blazor/hosting-models>.

   6 \. Appuyez sur **F5** pour exécuter l'application.

   > [!NOTE]
   > Si vous avez installé l’extension Visual Studio éblouissant pour une version préliminaire antérieure de ASP.NET Core éblouissant (version préliminaire 6 ou antérieure), vous pouvez désinstaller l’extension. L’installation des modèles éblouissants dans un interpréteur de commandes est désormais suffisante pour faire apparaître les modèles dans Visual Studio.

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1 \. Installez [Visual Studio Code](https://code.visualstudio.com/).

   2 \. Installez le dernier [ C# Visual Studio code extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp).

   3 \. Pour une expérience de webassembly éblouissant, exécutez la commande suivante dans une interface de commande :

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      Pour une expérience de serveur éblouissant, exécutez la commande suivante dans une interface de commande :

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      Pour plus d’informations sur les deux modèles d’hébergement éblouissants, le *serveur éblouissant* et le *webassembly éblouissant*, consultez <xref:blazor/hosting-models>.

   4 \. Ouvrez le dossier *WebApplication1* dans Visual Studio code.

   5 \. Pour un projet de serveur éblouissant, l’IDE demande que vous ajoutiez des ressources pour générer et déboguer le projet. Sélectionnez **Oui**.

   6 \. Si vous utilisez une application de serveur éblouissant, exécutez l’application à l’aide du débogueur Visual Studio Code. Si vous utilisez une application de webassembly éblouissante, exécutez `dotnet run` à partir du dossier du projet de l’application.

   7 \. Dans un navigateur, accédez à `https://localhost:5001`.

   <!--

   # [Visual Studio for Mac](#tab/visual-studio-mac)

   1\. Install [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/). Switch the [Update channel to Preview](/visualstudio/mac/install-preview).

   2\. Select **File** > **New Solution** or **New Project**.

   3\. In the sidebar, select **.NET Core** > **App**.

   4\. For a Blazor Server experience, select the **Blazor Server App** template. For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. For information on the two Blazor hosting models, *Blazor Server* and *Blazor WebAssembly*, see <xref:blazor/hosting-models>.

   5\. The **Target Framework** defaults to **.NET Core 3.0**. Select **Next**.

   6\. In the **Project Name** field, enter `WebApplication1`. Select **Create**.

   7\. Select **Run** > **Run Without Debugging** to run the app *without the debugger*. Running with the debugger isn't supported at this time.

   -->

   # <a name="net-core-clitabnetcore-cli"></a>[CLI .NET Core](#tab/netcore-cli/)

   Pour une expérience de webassembly éblouissant, exécutez les commandes suivantes dans une interface de commande :

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Pour une expérience de serveur éblouissant, exécutez les commandes suivantes dans une interface de commande :

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Pour plus d’informations sur les deux modèles d’hébergement éblouissants, le *serveur éblouissant* et le *webassembly éblouissant*, consultez <xref:blazor/hosting-models>.

   Dans un navigateur, accédez à `https://localhost:5001`.

   ---

::: moniker-end

::: moniker range="< aspnetcore-3.1"

1. Installez la dernière version du [Kit de développement logiciel (SDK) .net Core 3,0](https://dotnet.microsoft.com/download/dotnet-core/3.0) .

1. Installez éventuellement le modèle de [Webassembly éblouissant](xref:blazor/hosting-models#blazor-webassembly) en installant le [Kit de développement logiciel (SDK) .net Core 3,1 preview](https://dotnet.microsoft.com/download/dotnet-core/3.1) , puis en exécutant la commande suivante dans une interface de commande :

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview1.19508.20
   ```

1. Suivez les instructions de votre choix d’outils :

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1 \. Installez la dernière version de [Visual Studio](https://visualstudio.com/vs/) avec la charge de travail **développement Web et ASP.net** .

   2 \. Si vous le souhaitez, vous pouvez installer [Visual Studio 16,4 Preview 2 ou version ultérieure](https://visualstudio.microsoft.com/vs/preview/) avec la charge de travail **développement Web et ASP.net** pour le développement d’applications éblouissantes webassembly.

   3 \. Créer un nouveau projet.

   4 \. Sélectionnez l' **application éblouissant**. Sélectionnez **Suivant**.

   5 \. Indiquez un nom de projet dans le champ **Nom du projet**, ou acceptez le nom de projet par défaut. Confirmez que l’entrée d' **emplacement** est correcte ou indiquez un emplacement pour le projet. Sélectionnez **Créer**.

   6 \. Pour une expérience de webassembly éblouissant, choisissez le modèle **application éblouissant Webassembly** . Pour une expérience de serveur éblouissant, choisissez le modèle **application de serveur éblouissant** . Sélectionnez **Créer**. Pour plus d’informations sur les deux modèles d’hébergement éblouissants, le *serveur éblouissant* et le *webassembly éblouissant*, consultez <xref:blazor/hosting-models>.

   7 \. Appuyez sur **F5** pour exécuter l'application.

   > [!NOTE]
   > Si vous avez installé l’extension Visual Studio éblouissant pour une version préliminaire antérieure de ASP.NET Core éblouissant (version préliminaire 6 ou antérieure), vous pouvez désinstaller l’extension. L’installation des modèles éblouissants dans un interpréteur de commandes est désormais suffisante pour faire apparaître les modèles dans Visual Studio.

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1 \. Installez [Visual Studio Code](https://code.visualstudio.com/).

   2 \. Installez le dernier [ C# Visual Studio code extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp).

   3 \. Pour une expérience de webassembly éblouissant, exécutez la commande suivante dans une interface de commande :

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      Pour une expérience de serveur éblouissant, exécutez la commande suivante dans une interface de commande :

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      Pour plus d’informations sur les deux modèles d’hébergement éblouissants, le *serveur éblouissant* et le *webassembly éblouissant*, consultez <xref:blazor/hosting-models>.

   4 \. Ouvrez le dossier *WebApplication1* dans Visual Studio code.

   5 \. Pour un projet de serveur éblouissant, l’IDE demande que vous ajoutiez des ressources pour générer et déboguer le projet. Sélectionnez **Oui**.

   6 \. Si vous utilisez une application de serveur éblouissant, exécutez l’application à l’aide du débogueur Visual Studio Code. Si vous utilisez une application de webassembly éblouissante, exécutez `dotnet run` à partir du dossier du projet de l’application.

   7 \. Dans un navigateur, accédez à `https://localhost:5001`.

   <!--

   # [Visual Studio for Mac](#tab/visual-studio-mac)

   1\. Install [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/). Switch the [Update channel to Preview](/visualstudio/mac/install-preview).

   2\. Select **File** > **New Solution** or **New Project**.

   3\. In the sidebar, select **.NET Core** > **App**.

   4\. For a Blazor Server experience, select the **Blazor Server App** template. For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. For information on the two Blazor hosting models, *Blazor Server* and *Blazor WebAssembly*, see <xref:blazor/hosting-models>.

   5\. The **Target Framework** defaults to **.NET Core 3.0**. Select **Next**.

   6\. In the **Project Name** field, enter `WebApplication1`. Select **Create**.

   7\. Select **Run** > **Run Without Debugging** to run the app *without the debugger*. Running with the debugger isn't supported at this time.

   -->

   # <a name="net-core-clitabnetcore-cli"></a>[CLI .NET Core](#tab/netcore-cli/)

   Pour une expérience de webassembly éblouissant, exécutez les commandes suivantes dans une interface de commande :

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Pour une expérience de serveur éblouissant, exécutez les commandes suivantes dans une interface de commande :

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Pour plus d’informations sur les deux modèles d’hébergement éblouissants, le *serveur éblouissant* et le *webassembly éblouissant*, consultez <xref:blazor/hosting-models>.

   Dans un navigateur, accédez à `https://localhost:5001`.

   ---

::: moniker-end

Plusieurs pages sont disponibles à partir des onglets de la barre latérale :

* Début
* Counter
* Extraire les données

Sur la page Counter, sélectionnez le bouton **Click me** pour incrémenter le compteur sans actualisation de la page. L’incrémentation d’un compteur dans une page Web nécessite normalement l’écriture de JavaScript, mais avec C#un éblouissant que vous pouvez utiliser.

*Pages/Counter.razor* :

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter1.razor?highlight=7,12-15)]

Une demande de `/counter` dans le navigateur, comme spécifié par la directive `@page` en haut, entraîne le rendu du contenu par le composant `Counter`. Les composants sont rendus dans une représentation en mémoire de l’arborescence de rendu qui peut ensuite être utilisée pour mettre à jour l’interface utilisateur de manière flexible et efficace.

Chaque fois que le bouton **Click Me** est sélectionné :

* L’événement `onclick` est déclenché.
* La méthode `IncrementCount` est appelée.
* Le `currentCount` est incrémenté.
* Le composant est de nouveau restitué.

Le runtime compare le nouveau contenu au contenu précédent et applique uniquement le contenu modifié à l’Document Object Model (DOM).

Ajoutez un composant à un autre composant à l’aide de la syntaxe HTML. Par exemple, ajoutez le composant `Counter` à la page d’accueil de l’application en ajoutant un élément `<Counter />` au composant `Index`.

*Pages/Index.razor* :

[!code-cshtml[](get-started/samples_snapshot/3.x/Index1.razor?highlight=7)]

Exécuter l’application. La page d’accueil possède son propre compteur fourni par le composant `Counter`.

Les paramètres de composant sont spécifiés à l’aide d’attributs ou de [contenu enfant](xref:blazor/components#child-content), ce qui vous permet de définir des propriétés sur le composant enfant. Pour ajouter un paramètre au composant `Counter`, mettez à jour le bloc `@code` du composant :

* Ajoutez une propriété publique pour `IncrementAmount` avec un attribut `[Parameter]`.
* Modifiez la méthode `IncrementCount` pour utiliser `IncrementAmount` lorsque vous augmentez la valeur de `currentCount`.

*Pages/Counter.razor* :

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter2.razor?highlight=12-13,17)]

Spécifiez le `IncrementAmount` dans l’élément `<Counter>` `Index` du composant à l’aide d’un attribut.

*Pages/Index.razor* :

[!code-cshtml[](get-started/samples_snapshot/3.x/Index2.razor?highlight=7)]

Exécuter l’application. Le composant `Index` a son propre compteur qui est incrémenté de dix chaque fois que le bouton **Click Me** est sélectionné. Le composant `Counter` (*Counter. Razor*) à `/counter` continue d’être incrémenté d’une unité.

## <a name="next-steps"></a>Étapes suivantes

<xref:tutorials/first-blazor-app>

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:signalr/introduction>
