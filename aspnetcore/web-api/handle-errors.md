---
title: Gérer les erreurs dans les API Web ASP.NET Core
author: pranavkm
description: En savoir plus sur la gestion des erreurs avec ASP.NET Core API Web.
monikerRange: '>= aspnetcore-2.1'
ms.author: prkrishn
ms.custom: mvc
ms.date: 09/27/2019
uid: web-api/handle-errors
ms.openlocfilehash: dc21d4b2cf096b8d38b0a24d739e6874186004e7
ms.sourcegitcommit: 5d25a7f22c50ca6fdd0f8ecd8e525822e1b35b7a
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2019
ms.locfileid: "71551740"
---
# <a name="handle-errors-in-aspnet-core-web-apis"></a>Gérer les erreurs dans les API Web ASP.NET Core

Cet article explique comment gérer et personnaliser la gestion des erreurs avec ASP.NET Core API Web.

[Afficher ou télécharger l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/handle-errors/samples) ([Procédure de téléchargement](xref:index#how-to-download-a-sample))

## <a name="developer-exception-page"></a>Page d’exceptions du développeur

La [page exception du développeur](xref:fundamentals/error-handling) est un outil utile pour obtenir des traces de pile détaillées pour les erreurs de serveur. Il utilise <xref:Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware> pour capturer les exceptions synchrones et asynchrones à partir du pipeline HTTP et pour générer des réponses d’erreur. Pour illustrer ce propos, examinez l’action de contrôleur suivante :

[!code-csharp[](handle-errors/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_GetByCity)]

Exécutez la commande `curl` suivante pour tester l’action précédente :

```bash
curl -i https://localhost:5001/weatherforecast/chicago
```

::: moniker range=">= aspnetcore-3.0"

Dans ASP.NET Core 3,0 et versions ultérieures, la page exception du développeur affiche une réponse en texte brut si le client ne demande pas de sortie au format HTML. La sortie suivante apparaît :

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/plain
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:13:16 GMT

System.ArgumentException: We don't offer a weather forecast for chicago. (Parameter 'city')
   at WebApiSample.Controllers.WeatherForecastController.Get(String city) in C:\working_folder\aspnet\AspNetCore.Docs\aspnetcore\web-api\handle-errors\samples\3.x\Controllers\WeatherForecastController.cs:line 34
   at lambda_method(Closure , Object , Object[] )
   at Microsoft.Extensions.Internal.ObjectMethodExecutor.Execute(Object target, Object[] parameters)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ActionMethodExecutor.SyncObjectResultExecutor.Execute(IActionResultTypeMapper mapper, ObjectMethodExecutor executor, Object controller, Object[] arguments)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeActionMethodAsync>g__Logged|12_1(ControllerActionInvoker invoker)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeNextActionFilterAsync>g__Awaited|10_0(ControllerActionInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Rethrow(ActionExecutedContextSealed context)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Next(State& next, Scope& scope, Object& state, Boolean& isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.InvokeInnerFilterAsync()
--- End of stack trace from previous location where exception was thrown ---
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|19_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Logged|17_1(ResourceInvoker invoker)
   at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
   at Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
   at Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)

HEADERS
=======
Accept: */*
Host: localhost:44312
User-Agent: curl/7.55.1
```

Pour afficher une réponse au format HTML à la place, définissez l’en-tête de demande HTTP `Accept` sur le type de média `text/html`. Exemple :

```bash
curl -i -H "Accept: text/html" https://localhost:5001/weatherforecast/chicago
```

Examinez l’extrait suivant de la réponse HTTP :

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

Dans ASP.NET Core 2,2 et versions antérieures, la page exception du développeur affiche une réponse au format HTML. Par exemple, considérez l’extrait suivant de la réponse HTTP :

::: moniker-end

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:55:37 GMT

<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta charset="utf-8" />
        <title>Internal Server Error</title>
        <style>
            body {
    font-family: 'Segoe UI', Tahoma, Arial, Helvetica, sans-serif;
    font-size: .813em;
    color: #222;
    background-color: #fff;
}
```

::: moniker range=">= aspnetcore-3.0"

La réponse au format HTML devient utile lors des tests par le biais d’outils tels que poster. La capture d’écran suivante montre à la fois le texte brut et les réponses au format HTML dans le billet :

![Test de page d’exception de développeur dans le poster](handle-errors/_static/developer-exception-page-postman.gif)

::: moniker-end

> [!WARNING]
> Activez la page d’exception de développeur **uniquement quand l’application est en cours d’exécution dans l’environnement de développement**. Il n’est pas souhaitable de partager publiquement des informations détaillées sur les exceptions quand l’application s’exécute en production. Pour plus d’informations sur la configuration des environnements, consultez <xref:fundamentals/environments>.

## <a name="exception-handler"></a>Gestionnaire d’exceptions

Dans les environnements non-développement, l’intergiciel (middleware) de [gestion des exceptions](xref:fundamentals/error-handling) peut être utilisé pour générer une charge utile d’erreur :

1. Dans `Startup.Configure`, appelez <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*> pour utiliser l’intergiciel (middleware) :

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

1. Configurez une action de contrôleur pour `/error` répondre à l’itinéraire :

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

L’action `Error` précédente envoie une charge utile conforme à [RFC7807](https://tools.ietf.org/html/rfc7807)au client.

L’intergiciel (middleware) de gestion des exceptions peut également fournir une sortie de négociation de contenu plus détaillée dans l’environnement de développement local. Utilisez les étapes suivantes pour créer un format de charge utile cohérent dans les environnements de développement et de production :

1. Dans `Startup.Configure`, inscrivez les instances d’intergiciels (middleware) de gestion des exceptions spécifiques à l’environnement :

    ::: moniker range=">= aspnetcore-3.0"

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    Dans le code précédent, l’intergiciel est inscrit avec :

    * Itinéraire de dans `/error-local-development` l’environnement de développement.
    * Un itinéraire de `/error` dans des environnements qui ne sont pas des développements.
    
1. Appliquer le routage d’attribut aux actions du contrôleur :

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

## <a name="use-exceptions-to-modify-the-response"></a>Utiliser des exceptions pour modifier la réponse

Le contenu de la réponse peut être modifié à partir de l’extérieur du contrôleur. Dans l’API Web ASP.net 4. x, l’une des méthodes permettant d’effectuer <xref:System.Web.Http.HttpResponseException> cette opération était d’utiliser le type. ASP.NET Core n’inclut pas de type équivalent. La prise `HttpResponseException` en charge de peut être ajoutée avec les étapes suivantes :

1. Créez un type d’exception connu nommé `HttpResponseException`:

    [!code-csharp[](handle-errors/samples/3.x/Exceptions/HttpResponseException.cs?name=snippet_HttpResponseException)]

1. Créez un filtre d’action `HttpResponseExceptionFilter`nommé :

    [!code-csharp[](handle-errors/samples/3.x/Filters/HttpResponseExceptionFilter.cs?name=snippet_HttpResponseExceptionFilter)]

1. Dans `Startup.ConfigureServices`, ajoutez le filtre d’action à la collection de filtres :

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.2"
    
    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.1"

    [!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

## <a name="validation-failure-error-response"></a>Réponse d’erreur d’échec de validation

Pour les contrôleurs d’API Web, MVC <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> répond avec un type de réponse lors de l’échec de la validation du modèle. MVC utilise les résultats de <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.InvalidModelStateResponseFactory> pour construire la réponse d’erreur pour un échec de validation. L’exemple suivant utilise la fabrique pour modifier le type <xref:Microsoft.AspNetCore.Mvc.SerializableError> de réponse par défaut en in : `Startup.ConfigureServices`

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=4-13)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=5-14)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=6-15)]

::: moniker-end

## <a name="client-error-response"></a>Réponse d’erreur du client

Un *résultat d’erreur* est défini en tant que résultat avec le code d’état HTTP 400 ou une version ultérieure. Pour les contrôleurs d’API Web, MVC transforme un résultat d’erreur en <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>un résultat avec.

::: moniker range=">= aspnetcore-3.0"

La réponse d’erreur peut être configurée de l’une des manières suivantes :

1. [Implémenter ProblemDetailsFactory](#implement-problemdetailsfactory)
1. [Utilisez ApiBehaviorOptions. ClientErrorMapping](#use-apibehavioroptionsclienterrormapping)

### <a name="implement-problemdetailsfactory"></a>Implémenter ProblemDetailsFactory

MVC utilise `Microsoft.AspNetCore.Mvc.ProblemDetailsFactory` pour générer toutes les instances <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> de <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>et. Cela comprend les réponses d’erreur du client, les réponses d’erreur `Microsoft.AspNetCore.Mvc.ControllerBase.Problem` de <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem> validation et les méthodes d’assistance et.

Pour personnaliser la réponse des détails du problème, inscrivez une implémentation `ProblemDetailsFactory` personnalisée `Startup.ConfigureServices`de dans :

```csharp
public void ConfigureServices(IServiceCollection serviceCollection)
{
    services.AddControllers();
    services.AddTransient<ProblemDetailsFactory, CustomProblemDetailsFactory>();
}
```

::: moniker-end

::: moniker range="= aspnetcore-2.2"

La réponse d’erreur peut être configurée comme indiqué dans la section [use ApiBehaviorOptions. ClientErrorMapping](#use-apibehavioroptionsclienterrormapping) .

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="use-apibehavioroptionsclienterrormapping"></a>Utilisez ApiBehaviorOptions. ClientErrorMapping

Utilisez la propriété <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.ClientErrorMapping*> pour configurer le contenu de la réponse `ProblemDetails`. Par exemple, le code `Startup.ConfigureServices` suivant met à jour la propriété pour les `type` réponses 404 :

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=8-9)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=9-10)]

::: moniker-end
