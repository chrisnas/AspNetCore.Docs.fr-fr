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
# <a name="handle-errors-in-aspnet-core-web-apis"></a><span data-ttu-id="782a6-103">Gérer les erreurs dans les API Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="782a6-103">Handle errors in ASP.NET Core web APIs</span></span>

<span data-ttu-id="782a6-104">Cet article explique comment gérer et personnaliser la gestion des erreurs avec ASP.NET Core API Web.</span><span class="sxs-lookup"><span data-stu-id="782a6-104">This article describes how to handle and customize error handling with ASP.NET Core web APIs.</span></span>

<span data-ttu-id="782a6-105">[Afficher ou télécharger l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/handle-errors/samples) ([Procédure de téléchargement](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="782a6-105">[View or download sample code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/handle-errors/samples) ([How to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="developer-exception-page"></a><span data-ttu-id="782a6-106">Page d’exceptions du développeur</span><span class="sxs-lookup"><span data-stu-id="782a6-106">Developer Exception Page</span></span>

<span data-ttu-id="782a6-107">La [page exception du développeur](xref:fundamentals/error-handling) est un outil utile pour obtenir des traces de pile détaillées pour les erreurs de serveur.</span><span class="sxs-lookup"><span data-stu-id="782a6-107">The [Developer Exception Page](xref:fundamentals/error-handling) is a useful tool to get detailed stack traces for server errors.</span></span> <span data-ttu-id="782a6-108">Il utilise <xref:Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware> pour capturer les exceptions synchrones et asynchrones à partir du pipeline HTTP et pour générer des réponses d’erreur.</span><span class="sxs-lookup"><span data-stu-id="782a6-108">It uses <xref:Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware> to capture synchronous and asynchronous exceptions from the HTTP pipeline and to generate error responses.</span></span> <span data-ttu-id="782a6-109">Pour illustrer ce propos, examinez l’action de contrôleur suivante :</span><span class="sxs-lookup"><span data-stu-id="782a6-109">To illustrate, consider the following controller action:</span></span>

[!code-csharp[](handle-errors/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_GetByCity)]

<span data-ttu-id="782a6-110">Exécutez la commande `curl` suivante pour tester l’action précédente :</span><span class="sxs-lookup"><span data-stu-id="782a6-110">Run the following `curl` command to test the preceding action:</span></span>

```bash
curl -i https://localhost:5001/weatherforecast/chicago
```

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="782a6-111">Dans ASP.NET Core 3,0 et versions ultérieures, la page exception du développeur affiche une réponse en texte brut si le client ne demande pas de sortie au format HTML.</span><span class="sxs-lookup"><span data-stu-id="782a6-111">In ASP.NET Core 3.0 and later, the Developer Exception Page displays a plain-text response if the client doesn't request HTML-formatted output.</span></span> <span data-ttu-id="782a6-112">La sortie suivante apparaît :</span><span class="sxs-lookup"><span data-stu-id="782a6-112">The following output appears:</span></span>

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

<span data-ttu-id="782a6-113">Pour afficher une réponse au format HTML à la place, définissez l’en-tête de demande HTTP `Accept` sur le type de média `text/html`.</span><span class="sxs-lookup"><span data-stu-id="782a6-113">To display an HTML-formatted response instead, set the `Accept` HTTP request header to the `text/html` media type.</span></span> <span data-ttu-id="782a6-114">Exemple :</span><span class="sxs-lookup"><span data-stu-id="782a6-114">For example:</span></span>

```bash
curl -i -H "Accept: text/html" https://localhost:5001/weatherforecast/chicago
```

<span data-ttu-id="782a6-115">Examinez l’extrait suivant de la réponse HTTP :</span><span class="sxs-lookup"><span data-stu-id="782a6-115">Consider the following excerpt from the HTTP response:</span></span>

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

<span data-ttu-id="782a6-116">Dans ASP.NET Core 2,2 et versions antérieures, la page exception du développeur affiche une réponse au format HTML.</span><span class="sxs-lookup"><span data-stu-id="782a6-116">In ASP.NET Core 2.2 and earlier, the Developer Exception Page displays an HTML-formatted response.</span></span> <span data-ttu-id="782a6-117">Par exemple, considérez l’extrait suivant de la réponse HTTP :</span><span class="sxs-lookup"><span data-stu-id="782a6-117">For example, consider the following excerpt from the HTTP response:</span></span>

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

<span data-ttu-id="782a6-118">La réponse au format HTML devient utile lors des tests par le biais d’outils tels que poster.</span><span class="sxs-lookup"><span data-stu-id="782a6-118">The HTML-formatted response becomes useful when testing via tools like Postman.</span></span> <span data-ttu-id="782a6-119">La capture d’écran suivante montre à la fois le texte brut et les réponses au format HTML dans le billet :</span><span class="sxs-lookup"><span data-stu-id="782a6-119">The following screen capture shows both the plain-text and the HTML-formatted responses in Postman:</span></span>

![Test de page d’exception de développeur dans le poster](handle-errors/_static/developer-exception-page-postman.gif)

::: moniker-end

> [!WARNING]
> <span data-ttu-id="782a6-121">Activez la page d’exception de développeur **uniquement quand l’application est en cours d’exécution dans l’environnement de développement**.</span><span class="sxs-lookup"><span data-stu-id="782a6-121">Enable the Developer Exception Page **only when the app is running in the Development environment**.</span></span> <span data-ttu-id="782a6-122">Il n’est pas souhaitable de partager publiquement des informations détaillées sur les exceptions quand l’application s’exécute en production.</span><span class="sxs-lookup"><span data-stu-id="782a6-122">You don't want to share detailed exception information publicly when the app runs in production.</span></span> <span data-ttu-id="782a6-123">Pour plus d’informations sur la configuration des environnements, consultez <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="782a6-123">For more information on configuring environments, see <xref:fundamentals/environments>.</span></span>

## <a name="exception-handler"></a><span data-ttu-id="782a6-124">Gestionnaire d’exceptions</span><span class="sxs-lookup"><span data-stu-id="782a6-124">Exception handler</span></span>

<span data-ttu-id="782a6-125">Dans les environnements non-développement, l’intergiciel (middleware) de [gestion des exceptions](xref:fundamentals/error-handling) peut être utilisé pour générer une charge utile d’erreur :</span><span class="sxs-lookup"><span data-stu-id="782a6-125">In non-development environments, [Exception Handling Middleware](xref:fundamentals/error-handling) can be used to produce an error payload:</span></span>

1. <span data-ttu-id="782a6-126">Dans `Startup.Configure`, appelez <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*> pour utiliser l’intergiciel (middleware) :</span><span class="sxs-lookup"><span data-stu-id="782a6-126">In `Startup.Configure`, invoke <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*> to use the middleware:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

1. <span data-ttu-id="782a6-127">Configurez une action de contrôleur pour `/error` répondre à l’itinéraire :</span><span class="sxs-lookup"><span data-stu-id="782a6-127">Configure a controller action to respond to the `/error` route:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

<span data-ttu-id="782a6-128">L’action `Error` précédente envoie une charge utile conforme à [RFC7807](https://tools.ietf.org/html/rfc7807)au client.</span><span class="sxs-lookup"><span data-stu-id="782a6-128">The preceding `Error` action sends an [RFC7807](https://tools.ietf.org/html/rfc7807)-compliant payload to the client.</span></span>

<span data-ttu-id="782a6-129">L’intergiciel (middleware) de gestion des exceptions peut également fournir une sortie de négociation de contenu plus détaillée dans l’environnement de développement local.</span><span class="sxs-lookup"><span data-stu-id="782a6-129">Exception Handling Middleware can also provide more detailed content-negotiated output in the local development environment.</span></span> <span data-ttu-id="782a6-130">Utilisez les étapes suivantes pour créer un format de charge utile cohérent dans les environnements de développement et de production :</span><span class="sxs-lookup"><span data-stu-id="782a6-130">Use the following steps to produce a consistent payload format across development and production environments:</span></span>

1. <span data-ttu-id="782a6-131">Dans `Startup.Configure`, inscrivez les instances d’intergiciels (middleware) de gestion des exceptions spécifiques à l’environnement :</span><span class="sxs-lookup"><span data-stu-id="782a6-131">In `Startup.Configure`, register environment-specific Exception Handling Middleware instances:</span></span>

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

    <span data-ttu-id="782a6-132">Dans le code précédent, l’intergiciel est inscrit avec :</span><span class="sxs-lookup"><span data-stu-id="782a6-132">In the preceding code, the middleware is registered with:</span></span>

    * <span data-ttu-id="782a6-133">Itinéraire de dans `/error-local-development` l’environnement de développement.</span><span class="sxs-lookup"><span data-stu-id="782a6-133">A route of `/error-local-development` in the Development environment.</span></span>
    * <span data-ttu-id="782a6-134">Un itinéraire de `/error` dans des environnements qui ne sont pas des développements.</span><span class="sxs-lookup"><span data-stu-id="782a6-134">A route of `/error` in environments that aren't Development.</span></span>
    
1. <span data-ttu-id="782a6-135">Appliquer le routage d’attribut aux actions du contrôleur :</span><span class="sxs-lookup"><span data-stu-id="782a6-135">Apply attribute routing to controller actions:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

## <a name="use-exceptions-to-modify-the-response"></a><span data-ttu-id="782a6-136">Utiliser des exceptions pour modifier la réponse</span><span class="sxs-lookup"><span data-stu-id="782a6-136">Use exceptions to modify the response</span></span>

<span data-ttu-id="782a6-137">Le contenu de la réponse peut être modifié à partir de l’extérieur du contrôleur.</span><span class="sxs-lookup"><span data-stu-id="782a6-137">The contents of the response can be modified from outside of the controller.</span></span> <span data-ttu-id="782a6-138">Dans l’API Web ASP.net 4. x, l’une des méthodes permettant d’effectuer <xref:System.Web.Http.HttpResponseException> cette opération était d’utiliser le type.</span><span class="sxs-lookup"><span data-stu-id="782a6-138">In ASP.NET 4.x Web API, one way to do this was using the <xref:System.Web.Http.HttpResponseException> type.</span></span> <span data-ttu-id="782a6-139">ASP.NET Core n’inclut pas de type équivalent.</span><span class="sxs-lookup"><span data-stu-id="782a6-139">ASP.NET Core doesn't include an equivalent type.</span></span> <span data-ttu-id="782a6-140">La prise `HttpResponseException` en charge de peut être ajoutée avec les étapes suivantes :</span><span class="sxs-lookup"><span data-stu-id="782a6-140">Support for `HttpResponseException` can be added with the following steps:</span></span>

1. <span data-ttu-id="782a6-141">Créez un type d’exception connu nommé `HttpResponseException`:</span><span class="sxs-lookup"><span data-stu-id="782a6-141">Create a well-known exception type named `HttpResponseException`:</span></span>

    [!code-csharp[](handle-errors/samples/3.x/Exceptions/HttpResponseException.cs?name=snippet_HttpResponseException)]

1. <span data-ttu-id="782a6-142">Créez un filtre d’action `HttpResponseExceptionFilter`nommé :</span><span class="sxs-lookup"><span data-stu-id="782a6-142">Create an action filter named `HttpResponseExceptionFilter`:</span></span>

    [!code-csharp[](handle-errors/samples/3.x/Filters/HttpResponseExceptionFilter.cs?name=snippet_HttpResponseExceptionFilter)]

1. <span data-ttu-id="782a6-143">Dans `Startup.ConfigureServices`, ajoutez le filtre d’action à la collection de filtres :</span><span class="sxs-lookup"><span data-stu-id="782a6-143">In `Startup.ConfigureServices`, add the action filter to the filters collection:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.2"
    
    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.1"

    [!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

## <a name="validation-failure-error-response"></a><span data-ttu-id="782a6-144">Réponse d’erreur d’échec de validation</span><span class="sxs-lookup"><span data-stu-id="782a6-144">Validation failure error response</span></span>

<span data-ttu-id="782a6-145">Pour les contrôleurs d’API Web, MVC <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> répond avec un type de réponse lors de l’échec de la validation du modèle.</span><span class="sxs-lookup"><span data-stu-id="782a6-145">For web API controllers, MVC responds with a <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> response type when model validation fails.</span></span> <span data-ttu-id="782a6-146">MVC utilise les résultats de <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.InvalidModelStateResponseFactory> pour construire la réponse d’erreur pour un échec de validation.</span><span class="sxs-lookup"><span data-stu-id="782a6-146">MVC uses the results of <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.InvalidModelStateResponseFactory> to construct the error response for a validation failure.</span></span> <span data-ttu-id="782a6-147">L’exemple suivant utilise la fabrique pour modifier le type <xref:Microsoft.AspNetCore.Mvc.SerializableError> de réponse par défaut en in : `Startup.ConfigureServices`</span><span class="sxs-lookup"><span data-stu-id="782a6-147">The following example uses the factory to change the default response type to <xref:Microsoft.AspNetCore.Mvc.SerializableError> in `Startup.ConfigureServices`:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=4-13)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=5-14)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=6-15)]

::: moniker-end

## <a name="client-error-response"></a><span data-ttu-id="782a6-148">Réponse d’erreur du client</span><span class="sxs-lookup"><span data-stu-id="782a6-148">Client error response</span></span>

<span data-ttu-id="782a6-149">Un *résultat d’erreur* est défini en tant que résultat avec le code d’état HTTP 400 ou une version ultérieure.</span><span class="sxs-lookup"><span data-stu-id="782a6-149">An *error result* is defined as a result with an HTTP status code of 400 or higher.</span></span> <span data-ttu-id="782a6-150">Pour les contrôleurs d’API Web, MVC transforme un résultat d’erreur en <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>un résultat avec.</span><span class="sxs-lookup"><span data-stu-id="782a6-150">For web API controllers, MVC transforms an error result to a result with <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>.</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="782a6-151">La réponse d’erreur peut être configurée de l’une des manières suivantes :</span><span class="sxs-lookup"><span data-stu-id="782a6-151">The error response can be configured in one of the following ways:</span></span>

1. [<span data-ttu-id="782a6-152">Implémenter ProblemDetailsFactory</span><span class="sxs-lookup"><span data-stu-id="782a6-152">Implement ProblemDetailsFactory</span></span>](#implement-problemdetailsfactory)
1. [<span data-ttu-id="782a6-153">Utilisez ApiBehaviorOptions. ClientErrorMapping</span><span class="sxs-lookup"><span data-stu-id="782a6-153">Use ApiBehaviorOptions.ClientErrorMapping</span></span>](#use-apibehavioroptionsclienterrormapping)

### <a name="implement-problemdetailsfactory"></a><span data-ttu-id="782a6-154">Implémenter ProblemDetailsFactory</span><span class="sxs-lookup"><span data-stu-id="782a6-154">Implement ProblemDetailsFactory</span></span>

<span data-ttu-id="782a6-155">MVC utilise `Microsoft.AspNetCore.Mvc.ProblemDetailsFactory` pour générer toutes les instances <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> de <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>et.</span><span class="sxs-lookup"><span data-stu-id="782a6-155">MVC uses `Microsoft.AspNetCore.Mvc.ProblemDetailsFactory` to produce all instances of <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> and <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>.</span></span> <span data-ttu-id="782a6-156">Cela comprend les réponses d’erreur du client, les réponses d’erreur `Microsoft.AspNetCore.Mvc.ControllerBase.Problem` de <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem> validation et les méthodes d’assistance et.</span><span class="sxs-lookup"><span data-stu-id="782a6-156">This includes client error responses, validation failure error responses, and the `Microsoft.AspNetCore.Mvc.ControllerBase.Problem` and <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem> helper methods.</span></span>

<span data-ttu-id="782a6-157">Pour personnaliser la réponse des détails du problème, inscrivez une implémentation `ProblemDetailsFactory` personnalisée `Startup.ConfigureServices`de dans :</span><span class="sxs-lookup"><span data-stu-id="782a6-157">To customize the problem details response, register a custom implementation of `ProblemDetailsFactory` in `Startup.ConfigureServices`:</span></span>

```csharp
public void ConfigureServices(IServiceCollection serviceCollection)
{
    services.AddControllers();
    services.AddTransient<ProblemDetailsFactory, CustomProblemDetailsFactory>();
}
```

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<span data-ttu-id="782a6-158">La réponse d’erreur peut être configurée comme indiqué dans la section [use ApiBehaviorOptions. ClientErrorMapping](#use-apibehavioroptionsclienterrormapping) .</span><span class="sxs-lookup"><span data-stu-id="782a6-158">The error response can be configured as outlined in the [Use ApiBehaviorOptions.ClientErrorMapping](#use-apibehavioroptionsclienterrormapping) section.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="use-apibehavioroptionsclienterrormapping"></a><span data-ttu-id="782a6-159">Utilisez ApiBehaviorOptions. ClientErrorMapping</span><span class="sxs-lookup"><span data-stu-id="782a6-159">Use ApiBehaviorOptions.ClientErrorMapping</span></span>

<span data-ttu-id="782a6-160">Utilisez la propriété <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.ClientErrorMapping*> pour configurer le contenu de la réponse `ProblemDetails`.</span><span class="sxs-lookup"><span data-stu-id="782a6-160">Use the <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.ClientErrorMapping*> property to configure the contents of the `ProblemDetails` response.</span></span> <span data-ttu-id="782a6-161">Par exemple, le code `Startup.ConfigureServices` suivant met à jour la propriété pour les `type` réponses 404 :</span><span class="sxs-lookup"><span data-stu-id="782a6-161">For example, the following code in `Startup.ConfigureServices` updates the `type` property for 404 responses:</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=8-9)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=9-10)]

::: moniker-end