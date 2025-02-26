---
title: Autorisation basée sur les ressources dans ASP.NET Core
author: scottaddie
description: Apprenez à implémenter l’autorisation basée sur les ressources dans une application ASP.NET Core lorsqu’un attribut Authorize ne suffit pas.
ms.author: scaddie
ms.custom: mvc
ms.date: 11/15/2018
uid: security/authorization/resourcebased
ms.openlocfilehash: 835592521c714e270595e1448ae6e0aed1707b77
ms.sourcegitcommit: a166291c6708f5949c417874108332856b53b6a9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2019
ms.locfileid: "72590007"
---
# <a name="resource-based-authorization-in-aspnet-core"></a>Autorisation basée sur les ressources dans ASP.NET Core

La stratégie d’autorisation dépend de la ressource faisant l’objet d’un accès. Prenons l’exemple d’un document qui a une propriété auteur. Seul l’auteur est autorisé à mettre à jour le document. Par conséquent, le document doit être récupéré à partir du magasin de données avant que l’évaluation de l’autorisation puisse se produire.

L’évaluation d’attribut se produit avant la liaison de données et avant l’exécution du gestionnaire de page ou de l’action qui charge le document. Pour ces raisons, l’autorisation déclarative avec un attribut `[Authorize]` n’est pas suffisante. Au lieu de cela, vous pouvez appeler une méthode d’autorisation personnalisée &mdash;a style appelé *autorisation impérative*.

[Affichez ou téléchargez un exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/resourcebased/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample)).

[Créer une application ASP.net core avec des données utilisateur protégées par autorisation](xref:security/authorization/secure-data) contient un exemple d’application qui utilise l’autorisation basée sur les ressources.

## <a name="use-imperative-authorization"></a>Utiliser l’autorisation impérative

L’autorisation est implémentée en tant que service [IAuthorizationService](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice) et est inscrite dans la collection de services au sein de la classe `Startup`. Le service est rendu disponible via l' [injection de dépendances](xref:fundamentals/dependency-injection) aux gestionnaires de pages ou aux actions.

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp2/Controllers/DocumentController.cs?name=snippet_IAuthServiceDI&highlight=6)]

`IAuthorizationService` a deux surcharges de méthode `AuthorizeAsync` : l’une acceptant la ressource et le nom de la stratégie, l’autre acceptant la ressource et une liste de spécifications à évaluer.

::: moniker range=">= aspnetcore-2.0"

```csharp
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          IEnumerable<IAuthorizationRequirement> requirements);
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          string policyName);
```

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

```csharp
Task<bool> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          IEnumerable<IAuthorizationRequirement> requirements);
Task<bool> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          string policyName);
```

::: moniker-end

<a name="security-authorization-resource-based-imperative"></a>

Dans l’exemple suivant, la ressource à sécuriser est chargée dans un objet `Document` personnalisé. Une surcharge de `AuthorizeAsync` est appelée pour déterminer si l’utilisateur actuel est autorisé à modifier le document fourni. Une stratégie d’autorisation « EditPolicy » personnalisée est prise en compte dans la décision. Consultez [autorisation basée sur une stratégie personnalisée](xref:security/authorization/policies) pour plus d’informations sur la création de stratégies d’autorisation.

> [!NOTE]
> Les exemples de code suivants supposent que l’authentification s’exécute et définit la propriété `User`.

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp2/Pages/Document/Edit.cshtml.cs?name=snippet_DocumentEditHandler)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp1/Controllers/DocumentController.cs?name=snippet_DocumentEditAction)]

::: moniker-end

## <a name="write-a-resource-based-handler"></a>Écrire un gestionnaire basé sur des ressources

L’écriture d’un gestionnaire pour l’autorisation basée sur les ressources n’est pas très différente de celle de l' [écriture d’un gestionnaire de spécifications brutes](xref:security/authorization/policies#security-authorization-policies-based-authorization-handler). Créez une classe d’exigence personnalisée et implémentez une classe de gestionnaire des spécifications. Pour plus d’informations sur la création d’une classe d’exigence, consultez [spécifications](xref:security/authorization/policies#requirements).

La classe de gestionnaire spécifie la spécification et le type de ressource. Par exemple, un gestionnaire utilisant une `SameAuthorRequirement` et une ressource de `Document` suit :

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp2/Services/DocumentAuthorizationHandler.cs?name=snippet_HandlerAndRequirement)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp1/Services/DocumentAuthorizationHandler.cs?name=snippet_HandlerAndRequirement)]

::: moniker-end

Dans l’exemple précédent, imaginez que `SameAuthorRequirement` est un cas spécial d’une classe `SpecificAuthorRequirement` plus générique. La classe `SpecificAuthorRequirement` (non affichée) contient une propriété `Name` représentant le nom de l’auteur. La propriété `Name` peut avoir la valeur de l’utilisateur actuel.

Inscrire la spécification et le gestionnaire dans `Startup.ConfigureServices` :

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp2/Startup.cs?name=snippet_ConfigureServicesSample&highlight=3-7,9)]

### <a name="operational-requirements"></a>Exigences opérationnelles

Si vous prenez des décisions en fonction des résultats des opérations CRUD (création, lecture, mise à jour, suppression), utilisez la classe d’assistance [OperationAuthorizationRequirement](/dotnet/api/microsoft.aspnetcore.authorization.infrastructure.operationauthorizationrequirement) . Cette classe vous permet d’écrire un gestionnaire unique au lieu d’une classe individuelle pour chaque type d’opération. Pour l’utiliser, fournissez des noms d’opérations :

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp2/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_OperationsClass)]

Le gestionnaire est implémenté comme suit, à l’aide d’une `OperationAuthorizationRequirement` exigence et d’une ressource de `Document` :

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp2/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_Handler)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp1/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_Handler)]

::: moniker-end

Le gestionnaire précédent valide l’opération à l’aide de la ressource, de l’identité de l’utilisateur et de la propriété `Name` de l’exigence.

## <a name="challenge-and-forbid-with-an-operational-resource-handler"></a>Défi et interdisent avec un gestionnaire de ressources opérationnelles

Cette section montre comment le problème et les résultats de l’action interdire sont traités et comment les problèmes et les interdisent diffèrent.

Pour appeler un gestionnaire de ressources opérationnelles, spécifiez l’opération lors de l’appel de `AuthorizeAsync` dans votre gestionnaire de page ou action. L’exemple suivant détermine si l’utilisateur authentifié est autorisé à afficher le document fourni.

> [!NOTE]
> Les exemples de code suivants supposent que l’authentification s’exécute et définit la propriété `User`.

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp2/Pages/Document/View.cshtml.cs?name=snippet_DocumentViewHandler&highlight=10-11)]

Si l’autorisation est établie, la page d’affichage du document est retournée. Si l’autorisation échoue mais que l’utilisateur est authentifié, retourner `ForbidResult` informe tout intergiciel d’authentification qui a échoué. Une `ChallengeResult` est retournée lorsque l’authentification doit être effectuée. Pour les clients de navigateur interactifs, il peut être approprié de rediriger l’utilisateur vers une page de connexion.

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/ResourceBasedAuthApp1/Controllers/DocumentController.cs?name=snippet_DocumentViewAction&highlight=11-12)]

Si l’autorisation est réussie, la vue du document est retournée. Si l’autorisation échoue, le fait de retourner `ChallengeResult` informe l’intergiciel (middleware) d’authentification que l’autorisation a échoué et l’intergiciel (middleware) peut prendre la réponse appropriée. Une réponse appropriée peut retourner un code d’état 401 ou 403. Pour les clients de navigateur interactifs, cela peut signifier que vous redirigez l’utilisateur vers une page de connexion.

::: moniker-end
