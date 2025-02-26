---
title: Démarrage d’une application dans ASP.NET Core
author: rick-anderson
description: Découvrez comment la classe Startup en ASP.NET Core configure les services et le pipeline de requête de l’application.
ms.author: riande
ms.custom: mvc
ms.date: 8/7/2019
uid: fundamentals/startup
ms.openlocfilehash: 0ea3965f73f4b0334810bc9ec2910b0c9364a7ba
ms.sourcegitcommit: d8b12cc1716ee329d7bd2300e201b61e15d506ac
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/04/2019
ms.locfileid: "71942870"
---
# <a name="app-startup-in-aspnet-core"></a>Démarrage d’une application dans ASP.NET Core

Par [Rick Anderson](https://twitter.com/RickAndMSFT), [Tom Dykstra](https://github.com/tdykstra), [Luke Latham](https://github.com/guardrex) et [Steve Smith](https://ardalis.com)

La classe `Startup` configure les services et le pipeline de demande de l’application.

## <a name="the-startup-class"></a>Classe Startup

Les applications ASP.NET Core utilisent une classe `Startup`, appelée `Startup` par convention. La classe `Startup` :

* Inclut éventuellement une méthode <xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*> pour configurer les *services* de l’application. Un service est un composant réutilisable qui fournit une fonctionnalité d’application. Les services sont configurés &mdash;ou *inscrits*&mdash;dans `ConfigureServices` et consommés dans l’application via l’[injection de dépendances](xref:fundamentals/dependency-injection) ou <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices*>.
* Inclut une méthode <xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*> pour créer le pipeline de traitement des requêtes de l’application.

`ConfigureServices` et `Configure` sont appelés par le runtime ASP.NET Core au lancement de l’application :

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](startup/3.0_samples/StartupFilterSample/Startup.cs?name=snippet)]

L’exemple précédent concerne [Razor Pages](xref:razor-pages/index), toutefois, la version MVC est similaire.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](startup/sample_snapshot/Startup1.cs)]

::: moniker-end

La classe `Startup` est spécifiée quand l’[hôte](xref:fundamentals/index#host) de l’application est créé. La classe `Startup` est généralement spécifiée en appelant la méthode [`WebHostBuilderExtensions.UseStartup<TStartup>`](xref:Microsoft.AspNetCore.Hosting.WebHostBuilderExtensions.UseStartup*) dans le générateur d’hôte :

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](startup/sample_snapshot/Program3.cs?name=snippet_Program&highlight=12)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](startup/3.0_samples/Program3.cs?name=snippet_Program&highlight=12)]

L’hôte fournit des services accessibles au constructeur de classe `Startup`. L’application ajoute des services supplémentaires via `ConfigureServices`. Les services de l’hôte ainsi que ceux de l’application sont disponibles dans `Configure` et dans l’ensemble de l’application.

Seuls les types de service suivants peuvent être injectés dans le constructeur `Startup` lorsque vous utilisez <xref:Microsoft.Extensions.Hosting.IHostBuilder> :

* `IWebHostEnvironment`
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

[!code-csharp[](startup/3.0_samples/StartupFilterSample/StartUp2.cs?name=snippet)]

La plupart des services ne sont pas disponibles tant que la méthode `Configure` n’est pas appelée.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

L’hôte fournit des services accessibles au constructeur de classe `Startup`. L’application ajoute des services supplémentaires via `ConfigureServices`. Les services de l’hôte ainsi que ceux de l’application sont alors disponibles dans `Configure` et dans l’ensemble de l’application.

Une utilisation courante de [l’injection de dépendances](xref:fundamentals/dependency-injection) dans la classe `Startup` consiste à injecter :

* <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment> pour configurer les services en fonction de l’environnement.
* <xref:Microsoft.Extensions.Configuration.IConfiguration> pour lire la configuration.
* <xref:Microsoft.Extensions.Logging.ILoggerFactory> pour créer un enregistreur d’événements dans `Startup.ConfigureServices`.

[!code-csharp[](startup/sample_snapshot/Startup2.cs?highlight=7-8)]

La plupart des services ne sont pas disponibles tant que la méthode `Configure` n’est pas appelée.

::: moniker-end

### <a name="multiple-startup"></a>Démarrage multiple

Quand l’application définit différentes classes `Startup` pour différents environnements (par exemple, `StartupDevelopment`), la classe `Startup` appropriée est sélectionnée au moment de l’exécution. La classe dont le suffixe du nom correspond à l'environnement actuel est prioritaire. Si l’application est exécutée dans l’environnement de développement et comprend à la fois une classe `Startup` et une classe `StartupDevelopment`, la classe `StartupDevelopment` est utilisée. Pour plus d’informations, consultez [Utiliser plusieurs environnements](xref:fundamentals/environments#environment-based-startup-class-and-methods).

Pour plus d’informations sur l’hôte, consultez [L’hôte](xref:fundamentals/index#host). Pour plus d’informations sur la gestion des erreurs lors du démarrage, consultez [la gestion des exceptions de démarrage](xref:fundamentals/error-handling#startup-exception-handling).

## <a name="the-configureservices-method"></a>Méthode ConfigureServices

La méthode <xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*> est :

* facultatif.
* Appelée par l’hôte avant la méthode `Configure` pour configurer les services de l’application
* L’emplacement où les [options de configuration](xref:fundamentals/configuration/index) sont définies par convention.

L’hôte peut configurer certains services avant l’appel des méthodes `Startup`. Pour plus d’informations, consultez [L’hôte](xref:fundamentals/index#host).

Pour les fonctionnalités qui nécessitent une configuration importante, il existe des méthodes d’extension `Add{Service}` pour <xref:Microsoft.Extensions.DependencyInjection.IServiceCollection>. Par exemple,**Add**DbContext, **Add**DefaultIdentity, **Add**EntityFrameworkStores et **Add**RazorPages :

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](startup/3.0_samples/StartupFilterSample/StartupIdentity.cs?name=snippet)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](startup/sample_snapshot/Startup3.cs)]

::: moniker-end

L'ajout de services au conteneur de services les rend disponibles au sein de l’application et dans la méthode `Configure`. Les services sont résolus via l’[injection de dépendances](xref:fundamentals/dependency-injection) ou à partir de <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices*>.

::: moniker range="< aspnetcore-3.0"

Pour plus d’informations sur `SetCompatibilityVersion`, voir [SetCompatibilityVersion](xref:mvc/compatibility-version).

::: moniker-end

## <a name="the-configure-method"></a>Méthode Configure

La méthode <xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*> permet de spécifier le mode de réponse de l’application aux requêtes HTTP. Vous configurez le pipeline de requête en ajoutant des composants d’[intergiciel (middleware)](xref:fundamentals/middleware/index) à une instance de <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder>. `IApplicationBuilder` est disponible pour la méthode `Configure`, mais elle n’est pas inscrite dans le conteneur de service. L’hébergement crée un `IApplicationBuilder` et le passe directement à `Configure`.

Les [modèles ASP.NET Core](/dotnet/core/tools/dotnet-new) configurent le pipeline pour permettre la prise en charge des éléments suivants :

* [Page d’exceptions du développeur](xref:fundamentals/error-handling#developer-exception-page)
* [Gestionnaire d’exceptions](xref:fundamentals/error-handling#exception-handler-page)
* [HSTS (HTTP Strict Transport Security)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts)
* [Redirection HTTPS](xref:security/enforcing-ssl)
* [Fichiers statiques](xref:fundamentals/static-files)
* ASP.NET Core [MVC](xref:mvc/overview) et [Razor Pages](xref:razor-pages/index)

::: moniker range="< aspnetcore-3.0"

* [RGPD (règlement général sur la protection des données)](xref:security/gdpr)

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](startup/3.0_samples/StartupFilterSample/Startup.cs?name=snippet)]

L’exemple précédent concerne [Razor Pages](xref:razor-pages/index), toutefois, la version MVC est similaire.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](startup/sample_snapshot/Startup4.cs)]

::: moniker-end

Chaque méthode d’extension `Use` ajoute un ou plusieurs composants de middleware au pipeline de requête. Par exemple, <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles*> configure le [middleware](xref:fundamentals/middleware/index) pour qu’il traite des [fichiers statiques](xref:fundamentals/static-files).

Chaque composant de middleware du pipeline des requêtes est chargé d’appeler le composant suivant du pipeline ou de court-circuiter la chaîne, si c’est approprié.

::: moniker range=">= aspnetcore-3.0"

Vous pouvez spécifier des services supplémentaires (comme `IWebHostEnvironment`, `ILoggerFactory` et tout service défini dans `ConfigureServices`) dans la signature de la méthode `Configure`. Ces services sont injectés s’ils sont disponibles.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Vous pouvez spécifier des services supplémentaires (comme `IHostingEnvironment`, `ILoggerFactory` et tout service défini dans `ConfigureServices`) dans la signature de la méthode `Configure`. Ces services sont injectés s’ils sont disponibles.

::: moniker-end

Pour plus d’informations sur l’utilisation de `IApplicationBuilder` et sur l’ordre de traitement des middlewares, consultez <xref:fundamentals/middleware/index>.

<a name="convenience-methods"></a>

## <a name="configure-services-without-startup"></a>Configurer des services sans démarrage

Pour configurer les services et le pipeline de traitement de requête sans utiliser de classe `Startup`, appelez les méthodes pratiques `ConfigureServices` et `Configure` sur le générateur de l’hôte. Les appels multiples à `ConfigureServices` s’ajoutent les uns aux autres. S’il existe plusieurs appels de méthode `Configure`, le dernier appel de `Configure` est utilisé.

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](startup/3.0_samples/StartupFilterSample/Program1.cs?name=snippet)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](startup/sample_snapshot/Program1.cs?highlight=16,20)]

::: moniker-end

## <a name="extend-startup-with-startup-filters"></a>Étendre le démarrage avec les filtres de démarrage

Utilisez <xref:Microsoft.AspNetCore.Hosting.IStartupFilter> :

* Pour configurer l’intergiciel (middleware) au début ou à la fin du pipeline de [configuration](#the-configure-method) d’une application sans appel explicite à `Use{Middleware}`. `IStartupFilter` est utilisé par ASP.NET Core pour ajouter des valeurs par défaut au début du pipeline sans avoir à faire de l’auteur de l’application un enregistrement explicite de l’intergiciel (middleware) par défaut. `IStartupFilter` autorise un autre appel de composant `Use{Middleware}` pour le compte de l’auteur de l’application.
* Pour créer un pipeline de méthodes `Configure`. [IStartupFilter.Configure](xref:Microsoft.AspNetCore.Hosting.IStartupFilter.Configure*) peut définir un middleware à exécuter avant ou après les middlewares qui sont ajoutés par les bibliothèques.

`IStartupFilter` implémente <xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*>, qui reçoit et retourne un `Action<IApplicationBuilder>`. <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> définit une classe pour configurer le pipeline de requête d’une application. Pour plus d’informations, consultez [Créer un pipeline de middlewares avec IApplicationBuilder](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder).

Chaque `IStartupFilter` peut ajouter un ou plusieurs middlewares au pipeline de requête. Les filtres sont appelés dans l’ordre dans lequel ils ont été ajoutés au conteneur de service. Les filtres peuvent ajouter l’intergiciel avant ou après la transmission du contrôle au filtre suivant. Par conséquent, ils s’ajoutent au début ou à la fin du pipeline de l’application.

L’exemple suivant montre comment inscrire un middleware auprès de `IStartupFilter`. Le middleware `RequestSetOptionsMiddleware` définit une valeur d’options à partir d’un paramètre de chaîne de requête :

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](startup/3.0_samples/StartupFilterSample/RequestSetOptionsMiddleware.cs?name=snippet1)]

`RequestSetOptionsMiddleware` est configuré dans la classe `RequestSetOptionsStartupFilter` :

[!code-csharp[](startup/3.0_samples/StartupFilterSample/RequestSetOptionsStartupFilter.cs?name=snippet1&highlight=7)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](startup/sample_snapshot/RequestSetOptionsMiddleware.cs?name=snippet1&highlight=21)]

`RequestSetOptionsMiddleware` est configuré dans la classe `RequestSetOptionsStartupFilter` :

[!code-csharp[](startup/sample_snapshot/RequestSetOptionsStartupFilter.cs?name=snippet1&highlight=7)]

::: moniker-end

`IStartupFilter` est inscrit dans le conteneur de service de <xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*>.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](startup/3.0_samples/StartupFilterSample/Program.cs?name=snippet&highlight=19-20)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](startup/sample_snapshot/Program2.cs?name=snippet1&highlight=4-5)]

::: moniker-end

Quand un paramètre de chaîne de requête pour `option` est fourni, le middleware traite l’affectation de valeur avant que le middleware ASP.NET Core n’affiche la réponse.

L’ordre d’exécution de l'Intergiciel (middleware) est défini par l’ordre des inscriptions de `IStartupFilter` :

* Plusieurs implémentations d’`IStartupFilter` peuvent interagir avec les mêmes objets. Si l’ordre est important, définissez l’ordre des inscriptions de leurs services `IStartupFilter` pour qu’il corresponde à l’ordre dans lequel leurs intergiciels doivent s’exécuter.
* Les bibliothèques peuvent ajouter des intergiciels avec une ou plusieurs implémentations `IStartupFilter` qui s’exécutent avant ou après l’autre intergiciel d’application inscrit avec `IStartupFilter`. Pour appeler un middleware `IStartupFilter` avant un middleware ajouté par le `IStartupFilter` d’une bibliothèque :

  * Placez l’inscription du service avant l’ajout de la bibliothèque au conteneur de service.
  * Pour les appels suivants, placez l’inscription du service après l’ajout de la bibliothèque.

## <a name="add-configuration-at-startup-from-an-external-assembly"></a>Ajouter la configuration au démarrage à partir d’un assembly externe

Une implémentation de <xref:Microsoft.AspNetCore.Hosting.IHostingStartup> permet d’ajouter des améliorations à une application au démarrage à partir d’un assembly externe, en dehors de la classe `Startup` de l’application. Pour plus d'informations, consultez <xref:fundamentals/configuration/platform-specific-configuration>.

## <a name="additional-resources"></a>Ressources supplémentaires

* [L’hôte](xref:fundamentals/index#host)
* <xref:fundamentals/environments>
* <xref:fundamentals/middleware/index>
* <xref:fundamentals/logging/index>
* <xref:fundamentals/configuration/index>
