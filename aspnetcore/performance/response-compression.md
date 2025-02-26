---
title: Compression des réponses en ASP.NET Core
author: guardrex
description: Découvrez ce qu’est la compression des réponses et comment utiliser le middleware de compression des réponses dans les applications ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/09/2019
uid: performance/response-compression
ms.openlocfilehash: e320e87179f9f1b9773a55c380684a3f3f712632
ms.sourcegitcommit: 89fcc6cb3e12790dca2b8b62f86609bed6335be9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2019
ms.locfileid: "68993470"
---
# <a name="response-compression-in-aspnet-core"></a>Compression des réponses en ASP.NET Core

Par [Luke Latham](https://github.com/guardrex)

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

La bande passante réseau est une ressource limitée. Le fait de réduire la taille de la réponse augmente généralement la réactivité d’une application, parfois de manière considérable. Une façon de réduire la taille de la charge utile consiste à compresser les réponses de l’application.

## <a name="when-to-use-response-compression-middleware"></a>Quand utiliser l’intergiciel (middleware) de compression de réponse

Utilisez les technologies de compression de réponse basées sur le serveur dans IIS, Apache ou Nginx. Les performances de l’intergiciel (middleware) ne correspondront probablement pas à celles des modules de serveur. Le serveur de serveur [http. sys](xref:fundamentals/servers/httpsys) et le serveur [Kestrel](xref:fundamentals/servers/kestrel) n’offrent pas de prise en charge intégrée de la compression.

Utilisez l’intergiciel (middleware) de compression de réponse dans les cas suivants :

* Impossibilité d’utiliser les technologies de compression suivantes basées sur le serveur :
  * [Module de compression dynamique IIS](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Module Apache mod_deflate](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Décompression et compression Nginx](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* Hébergement direct:
  * [Serveur http. sys](xref:fundamentals/servers/httpsys) (anciennement appelé webListener)
  * [Serveur Kestrel](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>Compression des réponses

En règle générale, toute réponse qui n’est pas compressée en mode natif peut tirer parti de la compression des réponses. Les réponses qui ne sont pas compressées en mode natif sont généralement: CSS, JavaScript, HTML, XML et JSON. Vous ne devez pas compresser les ressources natives compressées, telles que les fichiers PNG. Si vous tentez de compresser davantage une réponse compressée en mode natif, toute petite réduction supplémentaire de taille et de temps de transmission sera probablement survenue du temps nécessaire au traitement de la compression. Ne compressez pas les fichiers d’une taille inférieure à environ 150-1000 octets (en fonction du contenu du fichier et de l’efficacité de la compression). La surcharge liée à la compression de petits fichiers peut entraîner la création d’un fichier compressé plus volumineux que le fichier non compressé.

Quand un client peut traiter du contenu compressé, il doit en informer le serveur en envoyant l'en-tête `Accept-Encoding` avec la requête. Quand un serveur envoie du contenu compressé, il doit inclure dans l’en-tête `Content-Encoding` des informations sur le mode d’encodage de la réponse compressée. Les désignations d'encodage de contenu prises en charge par l’intergiciel sont affichées dans le tableau suivant.

::: moniker range=">= aspnetcore-2.2"

| Valeurs d’en-tête `Accept-Encoding` | Intergiciel pris en charge | Description |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | Oui (valeur par défaut)        | [Format de données compressées Brotli](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | Non                   | [Format de données compressées compressé](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | Non                   | [Échange XML efficace W3C](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | Oui                  | [Format de fichier gzip](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | Oui                  | Identificateur «no Encoding»: La réponse ne doit pas être encodée. |
| `pack200-gzip`                  | Non                   | [Format de transfert réseau pour les archives Java](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | Oui                  | N'importe quel encodage de contenu disponible non explicitement demandé |

::: moniker-end

::: moniker range="< aspnetcore-2.2"

| Valeurs d’en-tête `Accept-Encoding` | Intergiciel pris en charge | Description |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | Non                   | [Format de données compressées Brotli](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | Non                   | [Format de données compressées compressé](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | Non                   | [Échange XML efficace W3C](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | Oui (valeur par défaut)        | [Format de fichier gzip](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | Oui                  | Identificateur «no Encoding»: La réponse ne doit pas être encodée. |
| `pack200-gzip`                  | Non                   | [Format de transfert réseau pour les archives Java](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | Oui                  | N'importe quel encodage de contenu disponible non explicitement demandé |

::: moniker-end

Pour plus d’informations, consultez la [liste IANA officielle des encodages de contenu](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry).

L’intergiciel vous permet d’ajouter des fournisseurs de compression supplémentaires pour des valeurs d’en-tête `Accept-Encoding` personnalisées. Pour plus d’informations, consultez [Fournisseurs personnalisés](#custom-providers) ci-dessous.

L’intergiciel peut réagir à la pondération de la valeur de qualité (qvalue, `q`) envoyée par le client pour hiérarchiser les schémas de compression. Pour plus d’informations, [consultez RFC 7231: Encodage](https://tools.ietf.org/html/rfc7231#section-5.3.4)Accept.

Les algorithmes de compression donnent lieu à un compromis entre vitesse de compression et efficacité de la compression. L’*Efficacité* dans ce contexte fait référence à la taille de la sortie après la compression. La plus petite taille est obtenue par la compression la plus *optimale*.

Les en-têtes impliqués dans la demande, l'envoi, la mise en cache et la réception de contenu compressé sont décrits dans le tableau ci-dessous.

| Header             | Rôle |
| ------------------ | ---- |
| `Accept-Encoding`  | Envoyé à partir du client au serveur pour indiquer les schémas d’encodage de contenu acceptables pour le client. |
| `Content-Encoding` | Envoyé à partir du serveur au client pour indiquer l'encodage du contenu de la charge utile. |
| `Content-Length`   | Au moment de la compression, l'en-tête `Content-Length` est supprimé car le contenu du corps change quand la réponse est compressée. |
| `Content-MD5`      | Durant la compression, l’en-tête `Content-MD5` est supprimé puisque le contenu du corps a changé et que le hachage n’est plus valide. |
| `Content-Type`     | Spécifie le type MIME du contenu. Chaque réponse doit spécifier son `Content-Type`. L’intergiciel vérifie cette valeur pour déterminer si la réponse doit être compressée. L’intergiciel (middleware) spécifie un ensemble de [types MIME par défaut](#mime-types) qu’il peut encoder, mais vous pouvez remplacer ou ajouter des types MIME. |
| `Vary`             | Quand l’en-tête `Vary` est envoyé par le serveur avec la valeur `Accept-Encoding` à des clients et proxys, ces derniers doivent mettre en cache les réponses (vary) en fonction de la valeur de l’en-tête `Accept-Encoding` de la requête. Si du contenu est retourné avec l'en-tête `Vary: Accept-Encoding`, les réponses (compressées et non compressées) sont mises en cache séparément. |

Explorez les fonctionnalités de l’intergiciel de compression des réponses avec l' [exemple d’application](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples). L’exemple illustre :

* La compression des réponses de l’application à l’aide de gzip et des fournisseurs de compression personnalisés.
* Comment ajouter un type MIME à la liste par défaut des types MIME pour la compression.

## <a name="package"></a>Package

::: moniker range=">= aspnetcore-3.0"

L’intergiciel (middleware) de compression des réponses est fourni par le package [Microsoft. AspNetCore. ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/) , qui est implicitement inclus dans les applications ASP.net core.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Pour inclure l’intergiciel (middleware) dans un projet, ajoutez une référence au AspNetCore [Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), qui inclut le package [Microsoft. ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/) .

::: moniker-end

## <a name="configuration"></a>Configuration

::: moniker range=">= aspnetcore-2.2"

Le code suivant montre comment activer l’intergiciel (middleware) de compression des réponses pour les types MIME et les fournisseurs de compression par défaut ([Brotli](#brotli-compression-provider) et [gzip](#gzip-compression-provider)):

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Le code suivant montre comment activer l’intergiciel (middleware) de compression des réponses pour les types MIME par défaut et le [fournisseur de compression gzip](#gzip-compression-provider):

::: moniker-end

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

Remarques :

* `app.UseResponseCompression`doit être appelé avant `app.UseMvc`.
* Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler), [Firebug](https://getfirebug.com/)ou [postal](https://www.getpostman.com/) pour définir l’en- `Accept-Encoding` tête de la demande et étudier les en-têtes, la taille et le corps de la réponse.

Soumettez une requête à l’exemple d’application sans l’en-tête `Accept-Encoding` et observez que la réponse n’est pas compressée. Les en-têtes  `Content-Encoding` et `Vary` ne sont pas présents sur la réponse.

![Fenêtre Fiddler indiquant le résultat d’une requête sans l’en-tête Accept-Encoding. La réponse n’est pas compressée.](response-compression/_static/request-uncompressed.png)

::: moniker range=">= aspnetcore-2.2"

Envoyez une demande à l’exemple d’application avec `Accept-Encoding: br` l’en-tête (compression Brotli) et observez que la réponse est compressée. Les en-têtes `Content-Encoding` et `Vary` sont présents sur la réponse.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et la valeur br. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse. La réponse est compressée.](response-compression/_static/request-compressed-br.png)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Soumettez une requête à l’exemple d’application avec l’en-tête `Accept-Encoding: gzip` et observez que la réponse est compressée. Les en-têtes `Content-Encoding` et `Vary` sont présents sur la réponse.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et une valeur de gzip. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse. La réponse est compressée.](response-compression/_static/request-compressed.png)

::: moniker-end

## <a name="providers"></a>Fournisseurs

::: moniker range=">= aspnetcore-2.2"

### <a name="brotli-compression-provider"></a>Fournisseur de compression Brotli

Utilisez le <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProvider> pour compresser les réponses avec le [format de données compressées Brotli](https://tools.ietf.org/html/rfc7932).

Si aucun fournisseur de compression n’est explicitement ajouté <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>à:

* Le fournisseur de compression Brotli est ajouté par défaut au groupe de fournisseurs de compression avec le [fournisseur de compression gzip](#gzip-compression-provider).
* La compression prend par défaut la compression Brotli lorsque le format de données compressées Brotli est pris en charge par le client. Si Brotli n’est pas pris en charge par le client, la compression est définie par défaut sur gzip lorsque le client prend en charge la compression gzip.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

Le fournisseur de compression Brotoli doit être ajouté lors de l’ajout explicite de fournisseurs de compression:

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

Définissez le niveau de compression <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProviderOptions>avec. Par défaut, le fournisseur de compression Brotli a le niveau de compression le plus rapide ([CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel)), qui peut ne pas produire la compression la plus efficace. Si vous souhaitez obtenir la compression la plus efficace possible, configurez l’intergiciel (middleware) pour une compression optimale.

| Niveau de compression | Description |
| ----------------- | ----------- |
| [CompressionLevel.Fastest](xref:System.IO.Compression.CompressionLevel) | La compression doit se terminer aussi rapidement que possible, même si le résultat n’est pas compressé de manière optimale. |
| [CompressionLevel.NoCompression](xref:System.IO.Compression.CompressionLevel) | Aucune compression ne doit être effectuée. |
| [CompressionLevel.Optimal](xref:System.IO.Compression.CompressionLevel) | Les réponses doivent être compressées de façon optimale, même si la compression prend plus de temps. |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<BrotliCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

::: moniker-end

### <a name="gzip-compression-provider"></a>Fournisseur de compression gzip

Utilisez pour compresser les réponses avec le [format de fichier gzip.](https://tools.ietf.org/html/rfc1952) <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider>

Si aucun fournisseur de compression n’est explicitement ajouté <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>à:

::: moniker range=">= aspnetcore-2.2"

* Le fournisseur de compression gzip est ajouté par défaut au tableau de fournisseurs de compression avec le [fournisseur de compression Brotli](#brotli-compression-provider).
* La compression prend par défaut la compression Brotli lorsque le format de données compressées Brotli est pris en charge par le client. Si Brotli n’est pas pris en charge par le client, la compression est définie par défaut sur gzip lorsque le client prend en charge la compression gzip.

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* Le fournisseur de compression gzip est ajouté par défaut au tableau de fournisseurs de compression.
* La compression est définie par défaut sur gzip lorsque le client prend en charge la compression gzip.

::: moniker-end

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

Le fournisseur de compression gzip doit être ajouté lors de l’ajout explicite de fournisseurs de compression:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

::: moniker-end

Définissez le niveau de compression <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions>avec. Par défaut, le fournisseur de compression gzip a le niveau de compression le plus rapide ([CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel)), qui peut ne pas produire la compression la plus efficace. Si vous souhaitez obtenir la compression la plus efficace possible, configurez l’intergiciel (middleware) pour une compression optimale.

| Niveau de compression | Description |
| ----------------- | ----------- |
| [CompressionLevel.Fastest](xref:System.IO.Compression.CompressionLevel) | La compression doit se terminer aussi rapidement que possible, même si le résultat n’est pas compressé de manière optimale. |
| [CompressionLevel.NoCompression](xref:System.IO.Compression.CompressionLevel) | Aucune compression ne doit être effectuée. |
| [CompressionLevel.Optimal](xref:System.IO.Compression.CompressionLevel) | Les réponses doivent être compressées de façon optimale, même si la compression prend plus de temps. |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>Fournisseurs personnalisés

Créer des implémentations de compression <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider>personnalisées avec. <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*> représente le contenu que cet encodage `ICompressionProvider` produit. L’intergiciel utilise ces informations pour choisir le fournisseur en fonction de la liste spécifiée dans l'en-tête `Accept-Encoding` de la requête.

À l’aide de l’exemple d’application, le client envoie une requête avec l'en-tête `Accept-Encoding: mycustomcompression`. L’intergiciel utilise l’implémentation de compression personnalisée et retourne la réponse avec un en-tête `Content-Encoding: mycustomcompression`. Pour que cette implémentation fonctionne, le client doit pouvoir décompresser l’encodage personnalisé.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/3.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/2.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

::: moniker-end

Soumettez une requête à l’exemple d’application avec l’en-tête `Accept-Encoding: mycustomcompression` et observez les en-têtes de réponse. Les en-têtes `Vary` et `Content-Encoding` sont présents sur la réponse. Le corps de la réponse (non affiché) n’est pas compressé par l’exemple. Il n’y a pas d’implémentation de compression dans la classe `CustomCompressionProvider` de l’exemple. Toutefois, ce dernier montre où vous pouvez implémenter un tel algorithme de compression.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et une valeur de mycustomcompression. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse.](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>types MIME

L’intergiciel (middleware) spécifie un ensemble de types MIME par défaut pour la compression:

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

Remplacez ou ajoutez des types MIME par les options de l’intergiciel (middleware) de compression des réponses. Notez que les types MIME génériques, tels `text/*` que ne sont pas pris en charge. L’exemple d’application ajoute un type MIME `image/svg+xml` pour et compresse et sert le ASP.net Core image de bannière (*Banner. svg*).

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

::: moniker-end

## <a name="compression-with-secure-protocol"></a>Compression avec protocole sécurisé

Les réponses compressées sur des connexions sécurisées peuvent être contrôlées à l'aide de l'option `EnableForHttps` (désactivée par défaut). L’utilisation de la compression avec des pages générées dynamiquement peut mener à des problèmes de sécurité tels que les attaques [CRIME](https://wikipedia.org/wiki/CRIME_(security_exploit)) et [BREACH](https://wikipedia.org/wiki/BREACH_(security_exploit)).

## <a name="adding-the-vary-header"></a>Ajout de l’en-tête Vary

Lors de la compression des réponses basées sur `Accept-Encoding` l’en-tête, il existe potentiellement plusieurs versions compressées de la réponse et une version non compressée. Pour indiquer au client et aux caches proxy que plusieurs versions existent et doivent être stockées, l' `Vary` en-tête est ajouté `Accept-Encoding` avec une valeur. Dans ASP.net Core 2,0 ou version ultérieure, l’intergiciel ajoute automatiquement `Vary` l’en-tête lorsque la réponse est compressée.

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Problème au niveau de l’intergiciel s'il est derrière un proxy inverse Nginx

Quand une requête est transmise par Nginx, l'en-tête `Accept-Encoding` est supprimé. La suppression de `Accept-Encoding` l’en-tête empêche l’intergiciel de compresser la réponse. Pour plus d’informations, consultez [NGINX : Compression et décompression](https://www.nginx.com/resources/admin-guide/compression-and-decompression/). Ce problème est suivi par la [compression directe pour Nginx (ASPNET/BasicMiddleware \#123)](https://github.com/aspnet/BasicMiddleware/issues/123).

## <a name="working-with-iis-dynamic-compression"></a>Utilisation de la compression dynamique IIS

Si vous disposez d’un module de compression dynamique IIS configuré au niveau du serveur que vous souhaitez désactiver pour une application, désactivez le module avec un ajout au fichier *Web. config* . Pour plus d’informations, consultez [Désactivation de modules IIS](xref:host-and-deploy/iis/modules#disabling-iis-modules).

## <a name="troubleshooting"></a>Résolution de problèmes

Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler), [Firebug](https://getfirebug.com/) ou [Postman](https://www.getpostman.com/) pour définir l'en-tête `Accept-Encoding` de la requête et étudier les en-têtes, la taille et le corps de la réponse. Par défaut, l’intergiciel (middleware) de compression des réponses compresse les réponses qui remplissent les conditions suivantes:

::: moniker range=">= aspnetcore-2.2"

* L' `Accept-Encoding` en-tête est présent avec une `br`valeur `gzip`, `*`, ou un encodage personnalisé qui correspond à un fournisseur de compression personnalisé que vous avez établi. La valeur ne doit pas être `identity` ou avoir une valeur de qualité (qvalue, `q`) égale à 0 (zéro).
* Le type MIME (`Content-Type`) doit être défini et doit correspondre à un type MIME configuré sur <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>le.
* La requête ne doit pas inclure l'en-tête `Content-Range`.
* La requête doit utiliser un protocole non sécurisé (http), sauf si un protocole sécurisé (https) est configuré dans les options de l’intergiciel de compression de la réponse. *Notez le danger [décrit ci-dessus](#compression-with-secure-protocol) lors de l’activation de la compression de contenu sécurisée.*

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* L'en-tête `Accept-Encoding` est présent avec la valeur `gzip`, `*`, ou un encodage personnalisé qui correspond à un fournisseur de compression personnalisé que vous avez établi. La valeur ne doit pas être `identity` ou avoir une valeur de qualité (qvalue, `q`) égale à 0 (zéro).
* Le type MIME (`Content-Type`) doit être défini et doit correspondre à un type MIME configuré sur <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>le.
* La requête ne doit pas inclure l'en-tête `Content-Range`.
* La requête doit utiliser un protocole non sécurisé (http), sauf si un protocole sécurisé (https) est configuré dans les options de l’intergiciel de compression de la réponse. *Notez le danger [décrit ci-dessus](#compression-with-secure-protocol) lors de l’activation de la compression de contenu sécurisée.*

::: moniker-end

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Réseau des développeurs Mozilla: Accepter-encodage](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [Section RFC 7231 3.1.2.1: Codages de contenu](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230, section 4.2.3: Codage gzip](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [Spécification de format de fichier GZIP version 4,3](https://www.ietf.org/rfc/rfc1952.txt)
