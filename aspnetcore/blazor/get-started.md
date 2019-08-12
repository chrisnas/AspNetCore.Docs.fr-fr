---
title: Prise en main d’ASP.NET Core éblouissante
author: guardrex
description: Commencez avec éblouissant en créant une application éblouissant avec les outils de votre choix.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 07/23/2019
uid: blazor/get-started
ms.openlocfilehash: b4609858be43acf9d1b2d8be5eff4879fd56f49f
ms.sourcegitcommit: 051f068c78931432e030b60094c38376d64d013e
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/24/2019
ms.locfileid: "68948329"
---
# <a name="get-started-with-aspnet-core-blazor"></a>Prise en main d’ASP.NET Core éblouissante

Par [Daniel Roth](https://github.com/danroth27) et [Luke Latham](https://github.com/guardrex)

Prise en main de éblouissant:

1. Installez la dernière version du [Kit de développement logiciel (SDK) .net Core 3,0 Preview](https://dotnet.microsoft.com/download/dotnet-core/3.0) .

1. Installez les modèles éblouissants en exécutant la commande suivante dans une interface de commande:

   ```console
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.0.0-preview7.19365.7
   ```

1. Suivez les instructions de votre choix d’outils:

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1 \. Installez la dernière version [préliminaire de Visual Studio](https://visualstudio.com/vs/preview) avec la charge de travail **développement Web et ASP.net** .

   2 \. Créer un nouveau projet.

   3 \. Sélectionnez l' **application éblouissant**. Sélectionnez **Suivant**.

   4 \. Indiquez un nom de projet dans le champ **Nom du projet**, ou acceptez le nom de projet par défaut. Confirmez que l’entrée d' **emplacement** est correcte ou indiquez un emplacement pour le projet. Sélectionnez **Créer**.

   5 \. Pour une expérience éblouissante côté client, choisissez le modèle **éblouissant (côté client)** . Pour une expérience côté serveur éblouissant, choisissez le modèle **application de serveur éblouissant** . Sélectionnez **Créer**. Pour plus d’informations sur les deux modèles d’hébergement éblouissants, côté serveur et côté client, <xref:blazor/hosting-models>consultez.

   6 \. Appuyez sur **F5** pour exécuter l'application.

   > [!NOTE]
   > Si vous avez installé l’extension Visual Studio éblouissant pour une version préliminaire antérieure de ASP.NET Core éblouissant (version préliminaire 6 ou antérieure), vous pouvez désinstaller l’extension dans Preview 7. L’installation des modèles éblouissants dans un interpréteur de commandes est désormais suffisante pour faire apparaître les modèles dans Visual Studio.

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1 \. Installez [Visual Studio Code](https://code.visualstudio.com/).

   2 \. Installez le dernier [ C# Visual Studio code extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp).

   3 \. Pour une expérience éblouissante côté client, exécutez la commande suivante dans une interface de commande:

      ```console
      dotnet new blazor -o WebApplication1
      ```

      Pour une expérience côté serveur éblouissant, exécutez la commande suivante dans une interface de commande:

      ```console
      dotnet new blazorserverside -o WebApplication1
      ```

      Pour plus d’informations sur les deux modèles d’hébergement éblouissants, côté serveur et côté client, <xref:blazor/hosting-models>consultez.

   4 \. Ouvrez le dossier *WebApplication1* dans Visual Studio code.

   5 \. Pour un projet côté serveur éblouissant, l’IDE demande que vous ajoutiez des ressources pour générer et déboguer le projet. Sélectionnez **Oui**.

   6 \. Si vous utilisez une application côté serveur éblouissante, exécutez l’application à l’aide du débogueur Visual Studio Code. Si vous utilisez une application cliente éblouissant, exécutez `dotnet run` à partir du dossier du projet de l’application.

   7 \. Dans un navigateur, accédez à `https://localhost:5001`.

   <!--

   # [Visual Studio for Mac](#tab/visual-studio-mac)

   1\. Install [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/). Switch the [Update channel to Preview](/visualstudio/mac/install-preview).

   2\. Select **File** > **New Solution** or **New Project**.

   3\. In the sidebar, select **.NET Core** > **App**.

   4\. For a Blazor server-side experience, select the **ASP.NET Core Blazor Server App** template. For a Blazor client-side experience, select the **ASP.NET Core Blazor WebAssembly App** template. Select **Next**. For information on the two Blazor hosting models, server-side and client-side, see <xref:blazor/hosting-models>.

   5\. The **Target Framework** defaults to **.NET Core 3.0**. Select **Next**.

   6\. In the **Project Name** field, enter `WebApplication1`. Select **Create**.

   7\. Select **Run** > **Run Without Debugging** to run the app *without the debugger*. Running with the debugger isn't supported at this time.

   -->

   # <a name="net-core-clitabnetcore-cli"></a>[CLI .NET Core](#tab/netcore-cli/)

   Pour une expérience éblouissante côté client, exécutez les commandes suivantes dans un interpréteur de commandes:

   ```console
   dotnet new blazor -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Pour une expérience côté serveur éblouissant, exécutez les commandes suivantes dans une interface de commande:

   ```console
   dotnet new blazorserverside -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Pour plus d’informations sur les deux modèles d’hébergement éblouissants, côté serveur et côté client, <xref:blazor/hosting-models>consultez.

   Dans un navigateur, accédez à `https://localhost:5001`.

   ---

Plusieurs pages sont disponibles à partir des onglets de la barre latérale:

* Dossier de base
* Compteur
* Extraire les données

Sur la page Counter, sélectionnez le bouton **Click me** pour incrémenter le compteur sans actualisation de la page. L’incrémentation d’un compteur dans une page Web nécessite normalement l’écriture de JavaScript, mais les composants C#Razor offrent une meilleure approche à l’aide de.

*Pages/Counter.razor* :

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter1.razor?highlight=7,12-15)]

Une demande de `/counter` dans le navigateur, comme spécifié par la `@page` directive en haut, fait en sorte `Counter` que le composant restitue son contenu. Les composants sont rendus dans une représentation en mémoire de l’arborescence de rendu qui peut ensuite être utilisée pour mettre à jour l’interface utilisateur de manière flexible et efficace.

Chaque fois que le bouton **Click Me** est sélectionné:

* L' `onclick` événement est déclenché.
* La méthode `IncrementCount` est appelée.
* `currentCount` Est incrémenté.
* Le composant est de nouveau restitué.

Le runtime compare le nouveau contenu au contenu précédent et applique uniquement le contenu modifié à l’Document Object Model (DOM).

Ajoutez un composant à un autre composant à l’aide de la syntaxe HTML. Par exemple, ajoutez le `Counter` composant à la page d’accueil de l’application `<Counter />` en ajoutant un `Index` élément au composant.

*Pages/Index.razor* :

[!code-cshtml[](get-started/samples_snapshot/3.x/Index1.razor?highlight=7)]

Exécuter l’application. La page d’accueil possède son propre compteur fourni `Counter` par le composant.

Les paramètres de composant sont spécifiés à l’aide d’attributs ou de [contenu enfant](xref:blazor/components#child-content), ce qui vous permet de définir des propriétés sur le composant enfant. Pour ajouter un paramètre au `Counter` composant, mettez à jour le bloc du `@code` composant:

* Ajoutez une propriété pour `IncrementAmount` avec un `[Parameter]` attribut.
* Modifiez la méthode `IncrementCount` pour utiliser `IncrementAmount` lorsque vous augmentez la valeur de `currentCount`.

*Pages/Counter.razor* :

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter2.razor?highlight=12-13,17)]

Spécifiez `Index` `<Counter>` dans l’élément du composant à l’aide d’un attribut. `IncrementAmount`

*Pages/Index.razor* :

[!code-cshtml[](get-started/samples_snapshot/3.x/Index2.razor?highlight=7)]

Exécuter l’application. Le `Index` composant a son propre compteur qui est incrémenté de dix chaque fois que le bouton **Click Me** est sélectionné. Le `Counter` composant (*Counter.* `/counter` Razor) de continue à être incrémenté d’une unité.

## <a name="next-steps"></a>Étapes suivantes

<xref:tutorials/first-blazor-app>

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:signalr/introduction>