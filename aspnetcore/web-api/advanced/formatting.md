---
title: Mettre en forme les données des réponses dans l’API web ASP.NET Core
author: ardalis
description: Découvrez comment mettre en forme les données des réponses dans l’API web ASP.NET Core.
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 8/22/2019
uid: web-api/advanced/formatting
ms.openlocfilehash: 78fe620ea8fdd681a276253f77939bcb2a56ebb9
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391290"
---
# <a name="format-response-data-in-aspnet-core-web-api"></a>Mettre en forme les données des réponses dans l’API web ASP.NET Core

De [Rick Anderson](https://twitter.com/RickAndMSFT) et [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC prend en charge la mise en forme des données de réponse. Les données de réponse peuvent être formatées à l’aide de formats spécifiques ou en réponse au format demandé par le client.

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/formatting) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

## <a name="format-specific-action-results"></a>Résultats des actions spécifiques au format

Certains types de résultats d’action sont spécifiques à un format particulier, comme <xref:Microsoft.AspNetCore.Mvc.JsonResult> et <xref:Microsoft.AspNetCore.Mvc.ContentResult>. Les actions peuvent retourner des résultats mis en forme dans un format particulier, indépendamment des préférences du client. Par exemple, le retour de `JsonResult` retourne des données au format JSON. Le retour de `ContentResult` ou d’une chaîne retourne des données de chaîne au format texte brut.

Une action n’est pas requise pour retourner un type spécifique. ASP.NET Core prend en charge toute valeur de retour d’objet.  Les résultats des actions qui retournent des objets qui ne sont pas des types <xref:Microsoft.AspNetCore.Mvc.IActionResult> sont sérialisés à l’aide de l’implémentation de <xref:Microsoft.AspNetCore.Mvc.Formatters.IOutputFormatter> appropriée. Pour plus d'informations, consultez <xref:web-api/action-return-types>.

La méthode d’assistance intégrée <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> retourne des données au format JSON : [!code-csharp @ no__t-2

L’exemple de téléchargement retourne la liste des auteurs. À l’aide des outils de développement du navigateur F12 ou du [poster](https://www.getpostman.com/tools) avec le code précédent :

* L’en-tête de réponse contenant **Content-type :** `application/json; charset=utf-8` s’affiche.
* Les en-têtes de demande s’affichent. Par exemple, l’en-tête `Accept`. L’en-tête `Accept` est ignoré par le code précédent.

Pour retourner des données mises en forme en texte brut, utilisez <xref:Microsoft.AspNetCore.Mvc.ContentResult.Content> et le helper <xref:Microsoft.AspNetCore.Mvc.ContentResult.Content> :

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_about)]

Dans le code précédent, le `Content-Type` retourné est `text/plain`. Le retour d’une chaîne remet `Content-Type` de `text/plain` :

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_string)]

Pour les actions avec plusieurs types de retour, retournez `IActionResult`. Par exemple, le retour de codes d’état HTTP différents en fonction du résultat des opérations effectuées.

## <a name="content-negotiation"></a>Négociation de contenu

La négociation de contenu se produit lorsque le client spécifie un [en-tête Accept](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html). Le format par défaut utilisé par ASP.NET Core est [JSON](https://json.org/). La négociation de contenu est :

* Implémenté par <xref:Microsoft.AspNetCore.Mvc.ObjectResult>.
* Intégré aux résultats d’action spécifiques au code d’État retournés par les méthodes d’assistance. Les méthodes d’assistance des résultats d’action sont basées sur `ObjectResult`.

Quand un type de modèle est retourné, le type de retour est `ObjectResult`.

La méthode d’action suivante utilise les méthodes helper `Ok` et `NotFound` :

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_search)]

Par défaut, ASP.NET Core prend en charge les types de média `application/json`, `text/json` et `text/plain`. Les outils tels que [Fiddler](https://www.telerik.com/fiddler) ou [postal](https://www.getpostman.com/tools) peuvent définir l’en-tête de demande `Accept` pour spécifier le format de retour. Lorsque l’en-tête `Accept` contient un type pris en charge par le serveur, ce type est retourné. La section suivante montre comment ajouter des formateurs supplémentaires.

Les actions de contrôleur peuvent retourner des POCO (Plain Old CLR Objects). Lorsqu’un POCO est retourné, le runtime crée automatiquement un `ObjectResult` qui encapsule l’objet. Le client obtient l’objet sérialisé mis en forme. Si l’objet retourné est `null`, une réponse `204 No Content` est retournée.

Retour d’un type d’objet :

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_alias)]

Dans le code précédent, une demande d’alias d’auteur valide retourne une réponse `200 OK` avec les données de l’auteur. Une demande d’alias non valide retourne une réponse `204 No Content`.

### <a name="the-accept-header"></a>En-tête Accept

La *négociation* de contenu a lieu lorsqu’un en-tête `Accept` apparaît dans la demande. Quand une demande contient un en-tête Accept, ASP.NET Core :

* Énumère les types de médias dans l’en-tête Accept dans l’ordre de préférence.
* Tente de trouver un formateur capable de produire une réponse dans l’un des formats spécifiés.

Si aucun formateur trouvé pouvant satisfaire la demande du client, ASP.NET Core :

* Retourne `406 Not Acceptable` si <xref:Microsoft.AspNetCore.Mvc.MvcOptions> a été défini, ou-
* Tente de trouver le premier formateur qui peut produire une réponse.

Si aucun formateur n’est configuré pour le format demandé, le premier formateur qui peut mettre en forme l’objet est utilisé. Si aucun en-tête `Accept` n’apparaît dans la requête :

* Le premier formateur qui peut gérer l’objet est utilisé pour sérialiser la réponse.
* Aucune négociation n’a lieu. Le serveur détermine le format à retourner.

Si l’en-tête Accept contient `*/*`, l’en-tête est ignoré, sauf si `RespectBrowserAcceptHeader` a la valeur true sur <xref:Microsoft.AspNetCore.Mvc.MvcOptions>.

### <a name="browsers-and-content-negotiation"></a>Navigateurs et négociation de contenu

Contrairement aux clients d’API typiques, les navigateurs Web fournissent des en-têtes `Accept`. Le navigateur Web spécifie de nombreux formats, y compris des caractères génériques. Par défaut, lorsque le Framework détecte que la demande provient d’un navigateur :

* L’en-tête `Accept` est ignoré.
* Le contenu est renvoyé au format JSON, sauf s’il a été configuré autrement.

Cela offre une expérience plus cohérente entre les navigateurs lors de l’utilisation des API.

Pour configurer une application pour honorer les en-têtes Accept du navigateur, définissez <xref:Microsoft.AspNetCore.Mvc.MvcOptions.RespectBrowserAcceptHeader> sur `true` :

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end

### <a name="configure-formatters"></a>Configurer les formateurs

Les applications qui doivent prendre en charge des formats supplémentaires peuvent ajouter les packages NuGet appropriés et configurer la prise en charge. Il existe des formateurs distincts pour les entrées et pour les sorties. Les formateurs d’entrée sont utilisés par la [liaison de modèle](xref:mvc/models/model-binding). Les formateurs de sortie sont utilisés pour mettre en forme les réponses. Pour plus d’informations sur la création d’un formateur personnalisé, consultez [formateurs personnalisés](xref:web-api/advanced/custom-formatters).

::: moniker range=">= aspnetcore-3.0"

### <a name="add-xml-format-support"></a>Ajouter la prise en charge du format XML

Les formateurs XML implémentés à l’aide de <xref:System.Xml.Serialization.XmlSerializer> sont configurés en appelant <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> :

[!code-csharp[](./formatting/3.0sample/Startup.cs?name=snippet)]

Le code précédent sérialise les résultats à l’aide de `XmlSerializer`.

Lorsque vous utilisez le code précédent, les méthodes de contrôleur doivent retourner le format approprié en fonction de l’en-tête `Accept` de la requête.

### <a name="configure-systemtextjson-based-formatters"></a>Configurer des formateurs basés sur System.Text.Json

Les fonctionnalités pour les formateurs basés sur `System.Text.Json` peuvent être configurées avec `Microsoft.AspNetCore.Mvc.JsonOptions.SerializerOptions`.

```csharp
services.AddControllers().AddJsonOptions(options =>
{
    // Use the default property (Pascal) casing.
    options.SerializerOptions.PropertyNamingPolicy = null;

    // Configure a custom converter.
    options.SerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

Les options de sérialisation de sortie, en fonction de l’action, peuvent être configurées à l’aide de `JsonResult`. Exemple :

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerOptions
    {
        options.WriteIndented = true,
    });
}
```

### <a name="add-newtonsoftjson-based-json-format-support"></a>Ajouter la prise en charge du format JSON basé sur Newtonsoft.Json

Avant ASP.NET Core 3,0, les formateurs JSON utilisés par défaut ont été implémentés à l’aide du package `Newtonsoft.Json`. Dans ASP.NET Core 3.0 ou version ultérieure, les formateurs JSON par défaut sont basés sur `System.Text.Json`. La prise en charge des formateurs et des fonctionnalités `Newtonsoft.Json` est disponible en installant le package NuGet [Microsoft. AspNetCore. Mvc. NewtonsoftJson](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) et en le configurant dans `Startup.ConfigureServices`.

[!code-csharp[](./formatting/3.0sample/StartupNewtonsoftJson.cs?name=snippet)]

Certaines fonctionnalités peuvent ne pas fonctionner correctement avec les formateurs @no__t 1/-0 et nécessitent une référence aux formateurs basés sur @no__t 1. Continuez à utiliser les formateurs @no__t 1/-0 si l’application :

* Utilise les attributs `Newtonsoft.Json`. Par exemple, `[JsonProperty]` ou `[JsonIgnore]`.
* Personnalise les paramètres de sérialisation.
* S’appuie sur les fonctionnalités fournies par `Newtonsoft.Json`.
* Configure `Microsoft.AspNetCore.Mvc.JsonResult.SerializerSettings`. Avant ASP.NET Core 3.0, `JsonResult.SerializerSettings` accepte une instance de `JsonSerializerSettings` spécifique à `Newtonsoft.Json`.
* Génère une documentation [OpenAPI](<xref:tutorials/web-api-help-pages-using-swagger>).

Les fonctionnalités des formateurs @no__t 1/-0 peuvent être configurées à l’aide de `Microsoft.AspNetCore.Mvc.MvcNewtonsoftJsonOptions.SerializerSettings` :

```csharp
services.AddControllers().AddNewtonsoftJson(options =>
{
    // Use the default property (Pascal) casing
    options.SerializerSettings.ContractResolver = new DefaultContractResolver();

    // Configure a custom converter
    options.SerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

Les options de sérialisation de sortie, en fonction de l’action, peuvent être configurées à l’aide de `JsonResult`. Exemple :

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerSettings
    {
        options.Formatting = Formatting.Indented,
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

### <a name="add-xml-format-support"></a>Ajouter la prise en charge du format XML

La mise en forme XML requiert le package NuGet [Microsoft. AspNetCore. Mvc. Formatters. xml](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Formatters.Xml/) .

Les formateurs XML implémentés à l’aide de <xref:System.Xml.Serialization.XmlSerializer> sont configurés en appelant <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> :

[!code-csharp[](./formatting/sample/Startup.cs?name=snippet)]

Le code précédent sérialise les résultats à l’aide de `XmlSerializer`.

Lorsque vous utilisez le code précédent, les méthodes de contrôleur doivent retourner le format approprié en fonction de l’en-tête `Accept` de la requête.

::: moniker-end

### <a name="specify-a-format"></a>Spécifier un format

Pour restreindre les formats de réponse, appliquez le filtre [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) . Comme la plupart des [filtres](xref:mvc/controllers/filters), `[Produces]` peut être appliqué au niveau de l’action, du contrôleur ou de l’étendue globale :

[!code-csharp[](./formatting/3.0sample/Controllers/WeatherForecastController.cs?name=snippet)]

Le filtre [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) précédent :

* Force toutes les actions du contrôleur à retourner des réponses au format JSON.
* Si d’autres formateurs sont configurés et que le client spécifie un format différent, JSON est retourné.

Pour plus d’informations, consultez [filtres](xref:mvc/controllers/filters).

### <a name="special-case-formatters"></a>Formateurs de cas spéciaux

Certains cas spéciaux sont implémentés avec des formateurs intégrés. Par défaut, les types de retour `string` sont mis en forme au format *texte/brut* (*texte/html* si demandé via l’en-tête `Accept`). Ce comportement peut être supprimé en supprimant le <xref:Microsoft.AspNetCore.Mvc.Formatters.StringOutputFormatter>. Les formateurs sont supprimés de la méthode `ConfigureServices`. Les actions qui ont un type de retour d’objet de modèle retournent `204 No Content` lors du retour de `null`. Ce comportement peut être supprimé en supprimant le <xref:Microsoft.AspNetCore.Mvc.Formatters.HttpNoContentOutputFormatter>. Le code suivant supprime `StringOutputFormatter` et `HttpNoContentOutputFormatter`.

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupStringOutputFormatter.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupStringOutputFormatter.cs?name=snippet)]
::: moniker-end

Sans le `StringOutputFormatter`, les formats de retour `string` intégrés au formateur JSON. Si le formateur JSON intégré est supprimé et qu’un formateur XML est disponible, le formateur XML met en forme les types de retour `string`. Sinon, les types de retour `string` retournent `406 Not Acceptable`.

Sans `HttpNoContentOutputFormatter`, les objets null sont mis en forme avec le formateur configuré. Exemple :

* Le module de formatage JSON retourne une réponse avec un corps de `null`.
* Le formateur XML retourne un élément XML vide avec l’attribut `xsi:nil="true"` défini.

## <a name="response-format-url-mappings"></a>Mappages d’URL de format de réponse

Les clients peuvent demander un format particulier dans le cadre de l’URL, par exemple :

* Dans la chaîne de requête ou dans une partie du chemin d’accès.
* En utilisant une extension de fichier spécifique au format, par exemple. XML ou. JSON.

Le mappage du chemin de la requête doit être spécifié dans la route utilisée par l’API. Exemple :

[!code-csharp[](./formatting/sample/Controllers/ProductsController.cs?name=snippet)]

L’itinéraire précédent permet de spécifier le format demandé en tant qu’extension de fichier facultative. L’attribut [`[FormatFilter]`](xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute) vérifie l’existence de la valeur de format dans le `RouteData` et mappe le format de réponse au formateur approprié lors de la création de la réponse.

|           Route        |             Formateur              |
|------------------------|------------------------------------|
|   `/api/products/5`    |    Le formateur de sortie par défaut    |
| `/api/products/5.json` | Le formateur JSON (s’il est configuré) |
| `/api/products/5.xml`  | Le formateur XML (s’il est configuré)  |
