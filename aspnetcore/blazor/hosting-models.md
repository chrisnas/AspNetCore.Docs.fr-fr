---
title: ASP.NET Core les modèles d’hébergement éblouissants
author: guardrex
description: Comprendre les modèles d’hébergement côté client et côté serveur.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 07/01/2019
uid: blazor/hosting-models
ms.openlocfilehash: 64393e826cb17550085f468f5916fca55973908f
ms.sourcegitcommit: 89fcc6cb3e12790dca2b8b62f86609bed6335be9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2019
ms.locfileid: "68993388"
---
# <a name="aspnet-core-blazor-hosting-models"></a>ASP.NET Core les modèles d’hébergement éblouissants

Par [Daniel Roth](https://github.com/danroth27)

Blazor est un Framework Web conçu pour s’exécuter côté client dans le navigateur sur un Runtime.net basé sur [webassembly](https://webassembly.org/) (*Blazor côté client*) ou côté serveur dans ASP.net Core (*Blazor côté serveur*). Quel que soit le modèle d’hébergement, les modèles d’application et de composant *sont les mêmes*.

Pour créer un projet pour les modèles d’hébergement décrits dans cet article, <xref:blazor/get-started>consultez.

## <a name="client-side"></a>Côté client

Le modèle d’hébergement principal pour éblouissant s’exécute côté client dans le navigateur sur webassembly. L’application Blazor, ses dépendances et le runtime .NET sont téléchargés sur le navigateur. L’application est exécutée directement sur le thread d’interface utilisateur du navigateur. Les mises à jour de l’interface utilisateur et la gestion des événements se produisent dans le même processus. Les ressources de l’application sont déployées en tant que fichiers statiques sur un serveur Web ou un service qui prend en charge le contenu statique sur les clients.

![Éblouissant côté client: L’application éblouissant s’exécute sur un thread d’interface utilisateur dans le navigateur.](hosting-models/_static/client-side.png)

Pour créer une application éblouissant à l’aide du modèle d’hébergement côté client, utilisez le modèle d' **application éblouissante** webassembly ([dotnet New blazorwasm](/dotnet/core/tools/dotnet-new)).

Après avoir sélectionné le modèle **application éblouissant** webassembly, vous avez la possibilité de configurer l’application pour utiliser un serveur principal ASP.net core en activant la case à cocher **ASP.net Core hébergée** ([dotnet New blazorwasm--Hosted](/dotnet/core/tools/dotnet-new)). L’application ASP.NET Core sert l’application éblouissant aux clients. L’application Blazor côté client peut interagir avec le serveur sur le réseau à l’aide d’appels d’API Web ou [Signalr](xref:signalr/introduction).

Les modèles incluent le script *éblouissant. webassembly. js* qui gère les éléments suivants:

* Téléchargement du Runtime .NET, de l’application et des dépendances de l’application.
* Initialisation du runtime pour exécuter l’application.

Le modèle d’hébergement côté client offre plusieurs avantages:

* Il n’existe aucune dépendance côté serveur .NET. L’application fonctionne entièrement après son téléchargement sur le client.
* Les ressources et les fonctionnalités du client sont pleinement exploitées.
* Le travail est déchargé du serveur vers le client.
* Un serveur Web ASP.NET Core n’est pas requis pour héberger l’application. Les scénarios de déploiement sans serveur sont possibles (par exemple, pour servir l’application à partir d’un CDN).

L’hébergement côté client présente des inconvénients:

* L’application est limitée aux fonctionnalités du navigateur.
* Le matériel et le logiciel client compatibles (par exemple, la prise en charge de webassembly) sont requis.
* La taille du téléchargement est supérieure, et les applications prennent plus de temps à se charger.
* Le Runtime .NET et la prise en charge des outils sont moins matures. Par exemple, des limitations existent dans la prise en charge de [.NET standard](/dotnet/standard/net-standard) et le débogage.

## <a name="server-side"></a>Côté serveur

Avec le modèle d’hébergement côté serveur, l’application est exécutée sur le serveur à partir d’une application ASP.NET Core. Les mises à jour de l’interface utilisateur, la gestion des événements et les appels JavaScript sont gérés par le biais d’une connexion [SignalR](xref:signalr/introduction).

![Le navigateur interagit avec l’application (hébergée à l’intérieur d’une application ASP.NET Core) sur le serveur via une connexion Signalr.](hosting-models/_static/server-side.png)

Pour créer une application éblouissant à l’aide du modèle d’hébergement côté serveur, utilisez le modèle d' **application ASP.net Core éblouissante Server** ([dotnet New blazorserver](/dotnet/core/tools/dotnet-new)). L’application ASP.NET Core héberge l’application côté serveur et crée le point de terminaison Signalr où les clients se connectent.

L’application ASP.net Core fait référence à la `Startup` classe de l’application à ajouter:

* Services côté serveur.
* L’application vers le pipeline de traitement des demandes.

Le script&dagger; *éblouissant. Server. js* établit la connexion client. Il est de la responsabilité de l’application de conserver et de restaurer l’état de l’application en fonction des besoins (par exemple, en cas de perte de connexion réseau).

Le modèle d’hébergement côté serveur offre plusieurs avantages:

* La taille du téléchargement est beaucoup plus petite qu’une application côté client et l’application est chargée beaucoup plus rapidement.
* L’application tire pleinement parti des fonctionnalités du serveur, notamment de l’utilisation de toutes les API compatibles avec .NET Core.
* .NET Core sur le serveur est utilisé pour exécuter l’application, de sorte que les outils .NET existants, tels que le débogage, fonctionnent comme prévu.
* Les clients légers sont pris en charge. Par exemple, les applications côté serveur fonctionnent avec des navigateurs qui ne prennent pas en charge webassembly et sur des appareils à ressources restreintes.
* Le .NET/C# base de code de l’application, y compris le code du composant de l’application, n’est pas pris en charge par les clients.

L’hébergement côté serveur présente des inconvénients:

* Une latence plus élevée existe généralement. Chaque interaction utilisateur implique un tronçon réseau.
* Il n’existe aucune prise en charge hors connexion. Si la connexion cliente échoue, l’application cesse de fonctionner.
* L’évolutivité est difficile pour les applications avec de nombreux utilisateurs. Le serveur doit gérer plusieurs connexions clientes et gérer l’état du client.
* Un serveur de ASP.NET Core est requis pour servir l’application. Les scénarios de déploiement sans serveur ne sont pas possibles (par exemple, pour servir l’application à partir d’un CDN).

&dagger;Le script *éblouissant. Server. js* est pris en charge à partir d’une ressource incorporée dans le ASP.net Core Framework partagé.

### <a name="reconnection-to-the-same-server"></a>Reconnexion au même serveur

Les applications côté serveur éblouissantes nécessitent une connexion active Signalr au serveur. Si la connexion est perdue, l’application tente de se reconnecter au serveur. Tant que l’état du client est toujours en mémoire, la session client reprend sans perte d’État.
 
Lorsque le client détecte que la connexion a été perdue, une interface utilisateur par défaut est affichée à l’utilisateur pendant que le client tente de se reconnecter. En cas d’échec de la reconnexion, l’utilisateur a la possibilité de réessayer. Pour personnaliser l’interface utilisateur, définissez un élément `components-reconnect-modal` avec `id` comme dans la page Razor *_Host. cshtml* . Le client met à jour cet élément avec l’une des classes CSS suivantes en fonction de l’état de la connexion:
 
* `components-reconnect-show`&ndash; Affichez l’interface utilisateur pour indiquer que la connexion a été perdue et que le client tente de se reconnecter.
* `components-reconnect-hide`&ndash; Le client dispose d’une connexion active et masque l’interface utilisateur.
* `components-reconnect-failed`&ndash; Échec de la reconnexion. Pour tenter une nouvelle connexion, appelez `window.Blazor.reconnect()`.

### <a name="stateful-reconnection-after-prerendering"></a>Reconnexion avec état après le prérendu
 
Les applications côté serveur éblouissantes sont configurées par défaut pour prérestituer l’interface utilisateur sur le serveur avant que la connexion cliente au serveur soit établie. Elle est configurée dans la page Razor *_Host. cshtml* :
 
```cshtml
<body>
    <app>@(await Html.RenderComponentAsync<App>())</app>
 
    <script src="_framework/blazor.server.js"></script>
</body>
```
 
Le client se reconnecte au serveur avec le même État que celui utilisé pour prégénérer l’application. Si l’état de l’application est toujours en mémoire, l’état du composant n’est pas restitué après l’établissement de la connexion Signalr.

### <a name="render-stateful-interactive-components-from-razor-pages-and-views"></a>Restituer des composants interactifs avec état à partir de pages et de vues Razor
 
Les composants interactifs avec état peuvent être ajoutés à une page ou à une vue Razor. Lorsque la page ou la vue est restituée, le composant est prérendu. L’application se reconnecte ensuite à l’état du composant une fois que la connexion cliente est établie tant que l’État est toujours en mémoire.
 
Par exemple, la page Razor suivante affiche un `Counter` composant avec un nombre initial spécifié à l’aide d’un formulaire:
 
```cshtml
<h1>My Razor Page</h1>

<form>
    <input type="number" asp-for="InitialCount" />
    <button type="submit">Set initial count</button>
</form>
 
@(await Html.RenderComponentAsync<Counter>(new { InitialCount = InitialCount }))
 
@code {
    [BindProperty(SupportsGet=true)]
    public int InitialCount { get; set; }
}
```

### <a name="detect-when-the-app-is-prerendering"></a>Détecter quand l’application est prérendu
 
[!INCLUDE[](~/includes/blazor-prerendering.md)]

### <a name="configure-the-signalr-client-for-blazor-server-side-apps"></a>Configurer le client Signalr pour les applications côté serveur éblouissantes
 
Parfois, vous devez configurer le client Signalr utilisé par les applications côté serveur éblouissantes. Par exemple, vous pouvez configurer la journalisation sur le client Signalr pour diagnostiquer un problème de connexion.
 
Pour configurer le client Signalr dans le fichier *pages/_Host. cshtml* :

* Ajoutez un `autostart="false"` attribut à la `<script>` balise du script *éblouissant. Server. js* .
* Appelez `Blazor.start` et transmettez un objet de configuration qui spécifie le générateur signalr.
 
```html
<script src="_framework/blazor.server.js" autostart="false"></script>
<script>
  Blazor.start({
    configureSignalR: function (builder) {
      builder.configureLogging(2); // LogLevel.Information
    }
  });
</script>
```

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:blazor/get-started>
* <xref:signalr/introduction>