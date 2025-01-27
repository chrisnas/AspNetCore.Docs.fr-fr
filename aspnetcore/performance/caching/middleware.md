---
title: Intergiciel de mise en cache des réponses dans ASP.NET Core
author: guardrex
description: Découvrez comment configurer et utiliser le middleware de mise en cache des réponses dans ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/09/2019
uid: performance/caching/middleware
ms.openlocfilehash: 838a08c12316d218501f26d5905f9e31ab93dfc9
ms.sourcegitcommit: 89fcc6cb3e12790dca2b8b62f86609bed6335be9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2019
ms.locfileid: "68994229"
---
# <a name="response-caching-middleware-in-aspnet-core"></a>Intergiciel de mise en cache des réponses dans ASP.NET Core

Par [Luke Latham](https://github.com/guardrex) et [John Luo](https://github.com/JunTaoLuo)

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/middleware/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

Cet article explique comment configurer l’intergiciel (middleware) de mise en cache des réponses dans une application ASP.NET Core. L’intergiciel détermine quand les réponses sont pouvant être mises en cache, stocke les réponses et sert les réponses du cache. Pour obtenir une présentation de la mise en cache HTTP et de l’attribut [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) , consultez [mise en cache des réponses](xref:performance/caching/response).

## <a name="configuration"></a>Configuration

::: moniker range=">= aspnetcore-3.0"

L’intergiciel (middleware) de mise en cache des réponses est rendu disponible par le package [Microsoft. AspNetCore. ResponseCaching](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/) , qui est implicitement ajouté aux applications ASP.net core.

Dans `Startup.ConfigureServices`, ajoutez l’intergiciel de mise en cache des réponses à la collection de services:

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

Configurez l’application pour utiliser l’intergiciel avec <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*> la méthode d’extension, qui ajoute l’intergiciel au pipeline de traitement `Startup.Configure`des demandes dans:

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=16)]

L’exemple d’application ajoute des en-têtes pour contrôler la mise en cache lors des requêtes suivantes:

* [Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2) &ndash; Met en cache les réponses pouvant être mises en cache pendant 10 secondes.
* [Faire varier](https://tools.ietf.org/html/rfc7231#section-7.1.4) Configure l’intergiciel (middleware) pour traiter une réponse mise en cache uniquement [`Accept-Encoding`](https://tools.ietf.org/html/rfc7231#section-5.3.4) si l’en-tête des demandes suivantes correspond à celui de la demande d’origine. &ndash;

[!code-csharp[](middleware/samples_snippets/3.x/AddHeaders.cs)]

L’intergiciel (middleware) de mise en cache des réponses met uniquement en cache les réponses du serveur qui génèrent un code d’état 200 (OK). Toutes les autres réponses, y compris les [pages d’erreur](xref:fundamentals/error-handling), sont ignorées par l’intergiciel (middleware).

> [!WARNING]
> Les réponses contenant du contenu pour les clients authentifiés doivent être marquées comme ne pouvant pas être mises en cache pour empêcher l’intergiciel de stocker et desservir ces réponses. Pour plus d’informations sur la façon dont l’intergiciel détermine si une réponse peut être mise en cache, consultez [conditions de mise en cache](#conditions-for-caching) .

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Utilisez le logiciel [Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) ou ajoutez une référence de package au package [Microsoft. AspNetCore. ResponseCaching](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/) .

Dans `Startup.ConfigureServices`, ajoutez l’intergiciel de mise en cache des réponses à la collection de services:

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

Configurez l’application pour utiliser l’intergiciel avec <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*> la méthode d’extension, qui ajoute l’intergiciel au pipeline de traitement `Startup.Configure`des demandes dans:

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=14)]

L’exemple d’application ajoute des en-têtes pour contrôler la mise en cache lors des requêtes suivantes:

* [Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2) &ndash; Met en cache les réponses pouvant être mises en cache pendant 10 secondes.
* [Faire varier](https://tools.ietf.org/html/rfc7231#section-7.1.4) Configure l’intergiciel (middleware) pour traiter une réponse mise en cache uniquement [`Accept-Encoding`](https://tools.ietf.org/html/rfc7231#section-5.3.4) si l’en-tête des demandes suivantes correspond à celui de la demande d’origine. &ndash;

[!code-csharp[](middleware/samples_snippets/2.x/AddHeaders.cs)]

L’intergiciel (middleware) de mise en cache des réponses met uniquement en cache les réponses du serveur qui génèrent un code d’état 200 (OK). Toutes les autres réponses, y compris les [pages d’erreur](xref:fundamentals/error-handling), sont ignorées par l’intergiciel (middleware).

> [!WARNING]
> Les réponses contenant du contenu pour les clients authentifiés doivent être marquées comme ne pouvant pas être mises en cache pour empêcher l’intergiciel de stocker et desservir ces réponses. Pour plus d’informations sur la façon dont l’intergiciel détermine si une réponse peut être mise en cache, consultez [conditions de mise en cache](#conditions-for-caching) .

::: moniker-end

## <a name="options"></a>Options

Les options de mise en cache des réponses sont indiquées dans le tableau suivant.

| Option | Description |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> | Plus grande taille pouvant être mise en cache pour le corps de la réponse en octets. La valeur par défaut `64 * 1024 * 1024` est (64 Mo). |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> | Taille limite de l’intergiciel (middleware) du cache de réponse en octets. La valeur par défaut `100 * 1024 * 1024` est (100 Mo). |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.UseCaseSensitivePaths> | Détermine si les réponses sont mises en cache sur les chemins d’accès sensibles à la casse. La valeur par défaut est `false`. |

L’exemple suivant configure l’intergiciel (middleware) sur:

* Met en cache les réponses dont la taille du corps est inférieure ou égale à 1 024 octets.
* Stockez les réponses en respectant les chemins sensibles à la casse. Par exemple, `/page1` et `/Page1` sont stockés séparément.

```csharp
services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024;
    options.UseCaseSensitivePaths = true;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

Lors de l’utilisation de contrôleurs d’API Web et MVC/Razor Pages des modèles de page, l’attribut [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) spécifie les paramètres nécessaires à la définition des en-têtes appropriés pour la mise en cache des réponses. Le seul paramètre de l' `[ResponseCache]` attribut qui requiert strictement l’intergiciel est <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute.VaryByQueryKeys>, qui ne correspond pas à un en-tête HTTP réel. Pour plus d'informations, consultez <xref:performance/caching/response#responsecache-attribute>.

Quand vous n’utilisez `[ResponseCache]` pas l’attribut, la mise en cache `VaryByQueryKeys`des réponses peut être modifiée avec. Utilisez le <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingFeature> directement à partir du [HttpContext. Features](xref:Microsoft.AspNetCore.Http.HttpContext.Features):

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();

if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

L’utilisation d’une valeur unique `*` égale `VaryByQueryKeys` à dans fait varier le cache par tous les paramètres de requête de demande.

## <a name="http-headers-used-by-response-caching-middleware"></a>En-têtes HTTP utilisés par l’intergiciel (middleware) de mise en cache des réponses

Le tableau suivant fournit des informations sur les en-têtes HTTP qui affectent la mise en cache des réponses.

| Header | Détails |
| ------ | ------- |
| `Authorization` | La réponse n’est pas mise en cache si l’en-tête existe. |
| `Cache-Control` | L’intergiciel (middleware) prend uniquement en compte les `public` réponses de mise en cache marquées avec la directive de cache. Contrôlez la mise en cache avec les paramètres suivants:<ul><li>âge maximal</li><li>Max-obsolète&#8224;</li><li>min-frais</li><li>doit être revalidate</li><li>non-cache</li><li>non-Store</li><li>uniquement-en cas de mise en cache</li><li>private</li><li>public</li><li>s-maxage</li><li>proxy-revalidate&#8225;</li></ul>&#8224;Si aucune limite n’est spécifiée `max-stale`à, l’intergiciel n’entreprend aucune action.<br>&#8225;`proxy-revalidate`a le même effet que `must-revalidate`.<br><br>Pour plus d’informations, [consultez RFC 7231: Demandez les directives](https://tools.ietf.org/html/rfc7234#section-5.2.1)de contrôle de cache. |
| `Pragma` | Un `Pragma: no-cache` en-tête dans la demande produit le même `Cache-Control: no-cache`effet que. Cet en-tête est substitué par les directives pertinentes dans `Cache-Control` l’en-tête, le cas échéant. Pris en compte pour la compatibilité descendante avec HTTP/1.0. |
| `Set-Cookie` | La réponse n’est pas mise en cache si l’en-tête existe. Tout intergiciel dans le pipeline de traitement des demandes qui définit un ou plusieurs cookies empêche l’intergiciel (middleware) de mise en cache des réponses de mettre en cache la réponse (par exemple, le [fournisseur TempData basé sur les cookies](xref:fundamentals/app-state#tempdata)).  |
| `Vary` | L' `Vary` en-tête est utilisé pour faire varier la réponse mise en cache par un autre en-tête. Par exemple, mettez en cache les réponses par encodage en incluant l’en-tête, qui met en cache `Accept-Encoding: text/plain` les `Vary: Accept-Encoding` réponses pour les demandes avec `Accept-Encoding: gzip` en-têtes et séparément. Une réponse avec une valeur d’en `*` -tête de n’est jamais stockée. |
| `Expires` | Une réponse considérée comme périmée par cet en-tête n’est pas stockée `Cache-Control` ou récupérée, sauf si elle est remplacée par d’autres en-têtes. |
| `If-None-Match` | La réponse complète est traitée à partir du cache si la `*` valeur n' `ETag` est pas et que la de la réponse ne correspond à aucune des valeurs fournies. Dans le cas contraire, une réponse 304 (non modifiée) est traitée. |
| `If-Modified-Since` | Si l' `If-None-Match` en-tête n’est pas présent, une réponse complète est traitée à partir du cache si la date de réponse mise en cache est plus récente que la valeur fournie. Dans le cas contraire, une réponse *304-non modifiée* est traitée. |
| `Date` | En cas de service à partir `Date` du cache, l’en-tête est défini par l’intergiciel (middleware) s’il n’a pas été fourni dans la réponse d’origine. |
| `Content-Length` | En cas de service à partir `Content-Length` du cache, l’en-tête est défini par l’intergiciel (middleware) s’il n’a pas été fourni dans la réponse d’origine. |
| `Age` | L' `Age` en-tête envoyé dans la réponse d’origine est ignoré. L’intergiciel (middleware) calcule une nouvelle valeur lors de la fourniture d’une réponse mise en cache. |

## <a name="caching-respects-request-cache-control-directives"></a>Mise en cache respecte les directives de contrôle de cache de demande

L’intergiciel respecte les règles de la [spécification de mise en cache HTTP 1,1](https://tools.ietf.org/html/rfc7234#section-5.2). Les règles requièrent qu’un cache honore `Cache-Control` un en-tête valide envoyé par le client. Dans le cadre de la spécification, un client peut faire `no-cache` des demandes avec une valeur d’en-tête et forcer le serveur à générer une nouvelle réponse pour chaque demande. Actuellement, il n’y a pas de contrôle du développeur sur ce comportement de mise en cache lors de l’utilisation de l’intergiciel (middleware), car l’intergiciel respecte la spécification officielle de mise en cache.

Pour plus de contrôle sur le comportement de mise en cache, explorez les autres fonctionnalités de mise en cache de ASP.NET Core. Consultez les rubriques suivantes :

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

## <a name="troubleshooting"></a>Résolution de problèmes

Si le comportement de mise en cache n’est pas le même que prévu, vérifiez que les réponses sont mises en cache et peuvent être servies à partir du cache. Examinez les en-têtes entrants de la demande et les en-têtes sortants de la réponse. Activez la [journalisation](xref:fundamentals/logging/index) pour faciliter le débogage.

Lors du test et du dépannage du comportement de mise en cache, un navigateur peut définir des en-têtes de demande qui affectent la mise en cache de façon indésirable. Par exemple, un navigateur peut définir l' `Cache-Control` en- `no-cache` tête `max-age=0` sur ou lors de l’actualisation d’une page. Les outils suivants peuvent définir explicitement des en-têtes de demande et sont préférés pour le test de mise en cache:

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>Conditions de mise en cache

* La demande doit aboutir à une réponse du serveur avec un code d’état 200 (OK).
* La méthode de demande doit être obtenir ou HEAD.
* Dans `Startup.Configure`, l’intergiciel de mise en cache des réponses doit être placé avant l’intergiciel (middleware) qui requiert la mise en cache. Pour plus d'informations, consultez <xref:fundamentals/middleware/index>.
* L' `Authorization` en-tête ne doit pas être présent.
* `Cache-Control`les paramètres d’en-tête doivent être valides et la `public` réponse doit être `private`marquée et non marquée.
* L' `Pragma: no-cache` en-tête ne doit pas être `Cache-Control` présent si l’en-tête `Cache-Control` n’est pas présent, `Pragma` car l’en-tête remplace l’en-tête lorsqu’il est présent.
* L' `Set-Cookie` en-tête ne doit pas être présent.
* `Vary`les paramètres d’en-tête doivent être valides et ne doivent pas être égaux à `*`.
* La `Content-Length` valeur d’en-tête (si définie) doit correspondre à la taille du corps de la réponse.
* Le <xref:Microsoft.AspNetCore.Http.Features.IHttpSendFileFeature> n’est pas utilisé.
* La réponse ne doit pas être périmée comme spécifié `Expires` par l’en `max-age` - `s-maxage` tête et les directives de cache et.
* La mise en mémoire tampon des réponses doit réussir. La taille de la réponse doit être inférieure à la valeur configurée <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit>ou par défaut. La taille du corps de la réponse doit être inférieure à la valeur configurée ou par défaut <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize>.
* La réponse doit être mise en cache conformément aux spécifications de la [norme RFC 7234](https://tools.ietf.org/html/rfc7234) . Par exemple, la `no-store` directive ne doit pas exister dans les champs d’en-tête de demande ou de réponse. Voir *la section 3: Stockage* des réponses dans les caches de [RFC 7234](https://tools.ietf.org/html/rfc7234) pour plus d’informations.

> [!NOTE]
> Le système anti-contrefaçon pour générer des jetons sécurisés afin d’empêcher les attaques par falsification de requête intersites ( `Cache-Control` CSRF `Pragma` ) définit les `no-cache` en-têtes et afin que les réponses ne soient pas mises en cache. Pour plus d’informations sur la façon de désactiver des jetons anti-contrefaçon pour les éléments <xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration>de formulaire HTML, consultez.

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
