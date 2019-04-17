---
ms.openlocfilehash: 41021e30ae85dd0ae42cbe6f1606727e21bd7707
ms.sourcegitcommit: 5995f44e9e13d7e7aa8d193e2825381c42184e47
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/02/2019
ms.locfileid: "58809386"
---
# <a name="aspnet-core-middleware-extensibility-sample"></a><span data-ttu-id="32417-101">Exemple d’extensibilité d’intergiciel (middleware) ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="32417-101">ASP.NET Core Middleware Extensibility Sample</span></span>

<span data-ttu-id="32417-102">Cet exemple illustre les scénarios décrits dans [Activation d’un intergiciel (middleware) basé sur une fabrique dans ASP.NET Core](https://docs.microsoft.com/aspnet/core/fundamentals/middleware/middleware-extensibility).</span><span class="sxs-lookup"><span data-stu-id="32417-102">This sample demonstrates the scenarios described in [Factory-based middleware activation in ASP.NET Core](https://docs.microsoft.com/aspnet/core/fundamentals/middleware/middleware-extensibility).</span></span>

<span data-ttu-id="32417-103">L’exemple d’application montre un middleware activé par :</span><span class="sxs-lookup"><span data-stu-id="32417-103">The sample app demonstrates middleware activated by:</span></span>

* <span data-ttu-id="32417-104">Convention.</span><span class="sxs-lookup"><span data-stu-id="32417-104">Convention.</span></span> <span data-ttu-id="32417-105">Pour plus d’informations sur l’activation conventionnelle de middleware, consultez la rubrique [Intergiciel (middleware)](https://docs.microsoft.com/aspnet/core/fundamentals/middleware/).</span><span class="sxs-lookup"><span data-stu-id="32417-105">For more information on conventional middleware activation, see the [Middleware](https://docs.microsoft.com/aspnet/core/fundamentals/middleware/) topic.</span></span>
* <span data-ttu-id="32417-106">Une implémentation de [IMiddleware](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.http.imiddleware).</span><span class="sxs-lookup"><span data-stu-id="32417-106">An [IMiddleware](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.http.imiddleware) implementation.</span></span> <span data-ttu-id="32417-107">La classe [IMiddlewareFactory](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.http.imiddlewarefactory) par défaut active le middleware.</span><span class="sxs-lookup"><span data-stu-id="32417-107">The default [IMiddlewareFactory](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.http.imiddlewarefactory) class activates the middleware.</span></span>

<span data-ttu-id="32417-108">Les implémentations de middleware fonctionnent de façon identique et enregistrent la valeur fournie par un paramètre de la chaîne de requête (`key`).</span><span class="sxs-lookup"><span data-stu-id="32417-108">The middleware implementations function identically and record the value provided by a query string parameter (`key`).</span></span> <span data-ttu-id="32417-109">Les middlewares utilisent un contexte de base de données injecté (un service délimité) pour enregistrer la valeur de la chaîne de requête dans une base de données en mémoire.</span><span class="sxs-lookup"><span data-stu-id="32417-109">The middlewares use an injected database context (a scoped service) to record the query string value in an in-memory database.</span></span>