---
title: Activation d’un intergiciel (middleware) basé sur une fabrique dans ASP.NET Core
author: guardrex
description: Découvrez comment utiliser un middleware fortement typé avec une implémentation de l’activation basée sur une fabrique dans ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/22/2019
uid: fundamentals/middleware/extensibility
ms.openlocfilehash: 17018d2dd20ed7b26bd0aa1095fa720a73f77261
ms.sourcegitcommit: d34b2627a69bc8940b76a949de830335db9701d3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2019
ms.locfileid: "71186944"
---
# <a name="factory-based-middleware-activation-in-aspnet-core"></a>Activation d’un intergiciel (middleware) basé sur une fabrique dans ASP.NET Core

Par [Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> est un point d’extensibilité pour l’activation d’un [middleware (intergiciel)](xref:fundamentals/middleware/index).

Les méthodes d’extension de <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> vérifient si le type inscrit d’un middleware implémente <xref:Microsoft.AspNetCore.Http.IMiddleware>. Si c’est le cas, l’instance de <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> inscrite dans le conteneur est utilisée pour résoudre l’implémentation de <xref:Microsoft.AspNetCore.Http.IMiddleware>, au lieu de la logique d’activation de middleware basée sur une convention. Le middleware est inscrit comme [service délimité ou temporaire](xref:fundamentals/dependency-injection#service-lifetimes) dans le conteneur de service de l’application.

Avantages :

* Activation par requête de client (injection de services délimités)
* Typage fort du middleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> est activé par requête de client (connexion) : des services délimités peuvent ainsi être injectés dans le constructeur du middleware.

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/middleware/extensibility/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> définit le middleware pour le pipeline des requêtes de l’application. La méthode [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) gère les requêtes et retourne un élément <xref:System.Threading.Tasks.Task> qui représente l’exécution du middleware.

Middleware activé par convention :

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

Middleware activé par <xref:Microsoft.AspNetCore.Http.MiddlewareFactory> :

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

Des extensions sont créées pour les middleware :

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

Il n’est pas possible de passer des objets au middleware activé par fabrique avec <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> :

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

Le middleware activé par fabrique est ajouté au conteneur intégré dans `Startup.ConfigureServices` :

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

Les deux middlewares sont inscrits dans le pipeline de traitement des requêtes dans `Startup.Configure` :

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=12-13)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> fournit des méthodes pour créer un middleware. L’implémentation de la fabrique de middlewares est inscrite dans le conteneur comme service délimité.

L’implémentation par défaut de <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, se trouve dans le package [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/).

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> est un point d’extensibilité pour l’activation d’un [middleware (intergiciel)](xref:fundamentals/middleware/index).

Les méthodes d’extension de <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> vérifient si le type inscrit d’un middleware implémente <xref:Microsoft.AspNetCore.Http.IMiddleware>. Si c’est le cas, l’instance de <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> inscrite dans le conteneur est utilisée pour résoudre l’implémentation de <xref:Microsoft.AspNetCore.Http.IMiddleware>, au lieu de la logique d’activation de middleware basée sur une convention. Le middleware est inscrit comme [service délimité ou temporaire](xref:fundamentals/dependency-injection#service-lifetimes) dans le conteneur de service de l’application.

Avantages :

* Activation par requête de client (injection de services délimités)
* Typage fort du middleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> est activé par requête de client (connexion) : des services délimités peuvent ainsi être injectés dans le constructeur du middleware.

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/middleware/extensibility/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> définit le middleware pour le pipeline des requêtes de l’application. La méthode [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) gère les requêtes et retourne un élément <xref:System.Threading.Tasks.Task> qui représente l’exécution du middleware.

Middleware activé par convention :

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

Middleware activé par <xref:Microsoft.AspNetCore.Http.MiddlewareFactory> :

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

Des extensions sont créées pour les middleware :

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

Il n’est pas possible de passer des objets au middleware activé par fabrique avec <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> :

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

Le middleware activé par fabrique est ajouté au conteneur intégré dans `Startup.ConfigureServices` :

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

Les deux middlewares sont inscrits dans le pipeline de traitement des requêtes dans `Startup.Configure` :

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=13-14)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> fournit des méthodes pour créer un middleware. L’implémentation de la fabrique de middlewares est inscrite dans le conteneur comme service délimité.

L’implémentation par défaut de <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, se trouve dans le package [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/).

::: moniker-end

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:fundamentals/middleware/index>
* <xref:fundamentals/middleware/extensibility-third-party-container>
