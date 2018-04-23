---
title: "Formateurs personnalisés dans les API web ASP.NET Core MVC"
author: tdykstra
description: "Découvrez comment créer et utiliser des formateurs personnalisés pour les API web dans ASP.NET Core."
manager: wpickett
ms.author: tdykstra
ms.date: 02/08/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: mvc/models/custom-formatters
ms.openlocfilehash: 8a42f2d885bd0a0c6d2bd05f9c589def2e15d50a
ms.sourcegitcommit: a510f38930abc84c4b302029d019a34dfe76823b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/30/2018
---
# <a name="custom-formatters-in-aspnet-core-mvc-web-apis"></a><span data-ttu-id="41d8f-103">Formateurs personnalisés dans les API web ASP.NET Core MVC</span><span class="sxs-lookup"><span data-stu-id="41d8f-103">Custom formatters in ASP.NET Core MVC web APIs</span></span>

<span data-ttu-id="41d8f-104">Par [Tom Dykstra](https://github.com/tdykstra)</span><span class="sxs-lookup"><span data-stu-id="41d8f-104">By [Tom Dykstra](https://github.com/tdykstra)</span></span>

<span data-ttu-id="41d8f-105">ASP.NET Core MVC prend en charge l’échange de données dans les API web à l’aide des formats JSON, XML ou texte brut.</span><span class="sxs-lookup"><span data-stu-id="41d8f-105">ASP.NET Core MVC has built-in support for data exchange in web APIs by using JSON, XML, or plain text formats.</span></span> <span data-ttu-id="41d8f-106">Cet article montre comment ajouter la prise en charge de formats supplémentaires en créant des formateurs personnalisés.</span><span class="sxs-lookup"><span data-stu-id="41d8f-106">This article shows how to add support for additional formats by creating custom formatters.</span></span>

<span data-ttu-id="41d8f-107">[Affichez ou téléchargez un exemple depuis GitHub](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-formatters/sample).</span><span class="sxs-lookup"><span data-stu-id="41d8f-107">[View or download sample from GitHub](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-formatters/sample).</span></span>

## <a name="when-to-use-custom-formatters"></a><span data-ttu-id="41d8f-108">Quand utiliser les formateurs personnalisés</span><span class="sxs-lookup"><span data-stu-id="41d8f-108">When to use custom formatters</span></span>

<span data-ttu-id="41d8f-109">Utilisez un formateur personnalisé quand vous souhaitez que le processus de [négociation de contenu](xref:mvc/models/formatting) prenne en charge un type de contenu non pris en charge par les formateurs intégrés (JSON, XML et texte brut).</span><span class="sxs-lookup"><span data-stu-id="41d8f-109">Use a custom formatter when you want the [content negotiation](xref:mvc/models/formatting) process to support a content type that isn't supported by the built-in formatters (JSON, XML, and plain text).</span></span>

<span data-ttu-id="41d8f-110">Par exemple, si certains clients de votre API web peuvent prendre en charge le format [Protobuf](https://github.com/google/protobuf), vous pouvez être amené à utiliser Protobuf avec ces clients, car il est plus efficace.</span><span class="sxs-lookup"><span data-stu-id="41d8f-110">For example, if some of the clients for your web API can handle the [Protobuf](https://github.com/google/protobuf) format, you might want to use Protobuf with those clients because it's more efficient.</span></span>  <span data-ttu-id="41d8f-111">Vous pouvez également demander à votre API web d’envoyer les noms et adresses des contacts au format [vCard](https://wikipedia.org/wiki/VCard), format couramment utilisé pour l’échange de données de contact.</span><span class="sxs-lookup"><span data-stu-id="41d8f-111">Or you might want your web API to send contact names and addresses in [vCard](https://wikipedia.org/wiki/VCard) format, a commonly used format for exchanging contact data.</span></span> <span data-ttu-id="41d8f-112">L’exemple d’application fourni dans cet article implémente un formateur vCard simple.</span><span class="sxs-lookup"><span data-stu-id="41d8f-112">The sample app provided with this article implements a simple vCard formatter.</span></span>

## <a name="overview-of-how-to-use-a-custom-formatter"></a><span data-ttu-id="41d8f-113">Présentation de l’utilisation d’un formateur personnalisé</span><span class="sxs-lookup"><span data-stu-id="41d8f-113">Overview of how to use a custom formatter</span></span>

<span data-ttu-id="41d8f-114">Voici les étapes à suivre pour créer et utiliser un formateur personnalisé :</span><span class="sxs-lookup"><span data-stu-id="41d8f-114">Here are the steps to create and use a custom formatter:</span></span>

* <span data-ttu-id="41d8f-115">Créez une classe de formateur de sortie si vous souhaitez sérialiser les données à envoyer au client.</span><span class="sxs-lookup"><span data-stu-id="41d8f-115">Create an output formatter class if you want to serialize data to send to the client.</span></span>
* <span data-ttu-id="41d8f-116">Créez une classe de formateur d’entrée si vous souhaitez désérialiser les données reçues en provenance du client.</span><span class="sxs-lookup"><span data-stu-id="41d8f-116">Create an input formatter class if you want to deserialize data received from the client.</span></span> 
* <span data-ttu-id="41d8f-117">Ajoutez des instances de vos formateurs aux collections `InputFormatters` et `OutputFormatters` dans [MvcOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.mvcoptions).</span><span class="sxs-lookup"><span data-stu-id="41d8f-117">Add instances of your formatters to the `InputFormatters` and `OutputFormatters` collections in [MvcOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.mvcoptions).</span></span>

<span data-ttu-id="41d8f-118">Les sections suivantes fournissent de l’aide et des exemples de code pour chacune de ces étapes.</span><span class="sxs-lookup"><span data-stu-id="41d8f-118">The following sections provide guidance and code examples for each of these steps.</span></span>

## <a name="how-to-create-a-custom-formatter-class"></a><span data-ttu-id="41d8f-119">Guide pratique pour créer une classe de formateur personnalisé</span><span class="sxs-lookup"><span data-stu-id="41d8f-119">How to create a custom formatter class</span></span>

<span data-ttu-id="41d8f-120">Pour créer un formateur :</span><span class="sxs-lookup"><span data-stu-id="41d8f-120">To create a formatter:</span></span>

* <span data-ttu-id="41d8f-121">Faites dériver la classe de la classe de base appropriée.</span><span class="sxs-lookup"><span data-stu-id="41d8f-121">Derive the class from the appropriate base class.</span></span>
* <span data-ttu-id="41d8f-122">Spécifiez les encodages et types de média valides dans le constructeur.</span><span class="sxs-lookup"><span data-stu-id="41d8f-122">Specify valid media types and encodings in the constructor.</span></span>
* <span data-ttu-id="41d8f-123">Substituez les méthodes `CanReadType`/`CanWriteType`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-123">Override `CanReadType`/`CanWriteType` methods</span></span>
* <span data-ttu-id="41d8f-124">Substituez les méthodes `ReadRequestBodyAsync`/`WriteResponseBodyAsync`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-124">Override `ReadRequestBodyAsync`/`WriteResponseBodyAsync` methods</span></span>
  
### <a name="derive-from-the-appropriate-base-class"></a><span data-ttu-id="41d8f-125">Effectuer une dérivation à partir de la classe de base appropriée</span><span class="sxs-lookup"><span data-stu-id="41d8f-125">Derive from the appropriate base class</span></span>

<span data-ttu-id="41d8f-126">Pour les médias de type texte (par exemple vCard), effectuez une dérivation à partir de la classe de base [TextInputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.textinputformatter) ou [TextOutputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.textoutputformatter).</span><span class="sxs-lookup"><span data-stu-id="41d8f-126">For text media types (for example, vCard), derive from the [TextInputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.textinputformatter) or [TextOutputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.textoutputformatter) base class.</span></span>

[!code-csharp[Main](custom-formatters/sample/Formatters/VcardOutputFormatter.cs?name=classdef)]

<span data-ttu-id="41d8f-127">Pour les types binaires, effectuez une dérivation à partir de la classe de base [InputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.inputformatter) ou [OutputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.outputformatter).</span><span class="sxs-lookup"><span data-stu-id="41d8f-127">For binary types, derive from the [InputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.inputformatter) or [OutputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.outputformatter) base class.</span></span>

### <a name="specify-valid-media-types-and-encodings"></a><span data-ttu-id="41d8f-128">Spécifier les encodages et types de média valides</span><span class="sxs-lookup"><span data-stu-id="41d8f-128">Specify valid media types and encodings</span></span>

<span data-ttu-id="41d8f-129">Dans le constructeur, spécifiez les encodages et types de média valides en effectuant les ajouts nécessaires aux collections `SupportedMediaTypes` et `SupportedEncodings`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-129">In the constructor, specify valid media types and encodings by adding to the `SupportedMediaTypes` and `SupportedEncodings` collections.</span></span>

[!code-csharp[Main](custom-formatters/sample/Formatters/VcardOutputFormatter.cs?name=ctor&highlight=3,5-6)]

> [!NOTE]  
> <span data-ttu-id="41d8f-130">Vous ne pouvez pas effectuer d’injection de dépendances de constructeur dans une classe de formateur.</span><span class="sxs-lookup"><span data-stu-id="41d8f-130">You can't do constructor dependency injection in a formatter class.</span></span> <span data-ttu-id="41d8f-131">Par exemple, vous ne pouvez pas obtenir un journaliseur en ajoutant un paramètre de journaliseur au constructeur.</span><span class="sxs-lookup"><span data-stu-id="41d8f-131">For example, you can't get a logger by adding a logger parameter to the constructor.</span></span> <span data-ttu-id="41d8f-132">Pour accéder aux services, vous devez utiliser l’objet de contexte passé à vos méthodes.</span><span class="sxs-lookup"><span data-stu-id="41d8f-132">To access services, you have to use the context object that gets passed in to your methods.</span></span> <span data-ttu-id="41d8f-133">L’exemple de code [ci-dessous](#read-write) montre comment procéder.</span><span class="sxs-lookup"><span data-stu-id="41d8f-133">A code example [below](#read-write) shows how to do this.</span></span>

### <a name="override-canreadtypecanwritetype"></a><span data-ttu-id="41d8f-134">Substituer CanReadType/CanWriteType</span><span class="sxs-lookup"><span data-stu-id="41d8f-134">Override CanReadType/CanWriteType</span></span> 

<span data-ttu-id="41d8f-135">Spécifiez le type cible de la désérialisation ou de la sérialisation en remplaçant les méthodes `CanReadType` ou `CanWriteType`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-135">Specify the type you can deserialize into or serialize from by overriding the `CanReadType` or `CanWriteType` methods.</span></span> <span data-ttu-id="41d8f-136">Par exemple, vous pouvez peut-être créer uniquement un texte vCard à partir d’un type `Contact`, et inversement.</span><span class="sxs-lookup"><span data-stu-id="41d8f-136">For example, you might only be able to create vCard text from a `Contact` type and vice versa.</span></span>

[!code-csharp[Main](custom-formatters/sample/Formatters/VcardOutputFormatter.cs?name=canwritetype)]

#### <a name="the-canwriteresult-method"></a><span data-ttu-id="41d8f-137">Méthode CanWriteResult</span><span class="sxs-lookup"><span data-stu-id="41d8f-137">The CanWriteResult method</span></span>

<span data-ttu-id="41d8f-138">Dans certains cas, vous devez substituer `CanWriteResult` au lieu de `CanWriteType`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-138">In some scenarios you have to override `CanWriteResult` instead of `CanWriteType`.</span></span> <span data-ttu-id="41d8f-139">Utilisez `CanWriteResult` si les conditions suivantes sont vraies :</span><span class="sxs-lookup"><span data-stu-id="41d8f-139">Use `CanWriteResult` if the following conditions are true:</span></span>

  * <span data-ttu-id="41d8f-140">Votre méthode d’action retourne une classe de modèle.</span><span class="sxs-lookup"><span data-stu-id="41d8f-140">Your action method returns a model class.</span></span>
  * <span data-ttu-id="41d8f-141">Il existe des classes dérivées qui peuvent être retournées au moment de l’exécution.</span><span class="sxs-lookup"><span data-stu-id="41d8f-141">There are derived classes which might be returned at runtime.</span></span>
  * <span data-ttu-id="41d8f-142">Vous devez savoir au moment de l’exécution quelle est la classe dérivée retournée par l’action.</span><span class="sxs-lookup"><span data-stu-id="41d8f-142">You need to know at runtime which derived class was returned by the action.</span></span>  

<span data-ttu-id="41d8f-143">Par exemple, la signature de votre méthode d’action retourne un type `Person`, mais elle peut éventuellement retourner un type `Student` ou un type `Instructor` dérivé de `Person`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-143">For example, suppose your action method signature returns a `Person` type, but it may return a `Student` or `Instructor` type that derives from `Person`.</span></span> <span data-ttu-id="41d8f-144">Si vous souhaitez que votre formateur gère uniquement les objets `Student`, vérifiez le type d’[Object](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.outputformattercanwritecontext#Microsoft_AspNetCore_Mvc_Formatters_OutputFormatterCanWriteContext_Object) dans l’objet de contexte fourni à la méthode `CanWriteResult`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-144">If you want your formatter to handle only `Student` objects, check the type of [Object](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.outputformattercanwritecontext#Microsoft_AspNetCore_Mvc_Formatters_OutputFormatterCanWriteContext_Object) in the context object provided to the `CanWriteResult` method.</span></span> <span data-ttu-id="41d8f-145">Notez qu’il n’est pas nécessaire d’utiliser `CanWriteResult` quand la méthode d’action retourne `IActionResult`. Dans ce cas, la méthode `CanWriteType` reçoit le type au moment de l’exécution.</span><span class="sxs-lookup"><span data-stu-id="41d8f-145">Note that it's not necessary to use `CanWriteResult` when the action method returns `IActionResult`; in that case, the `CanWriteType` method receives the runtime type.</span></span>

<a id="read-write"></a>
### <a name="override-readrequestbodyasyncwriteresponsebodyasync"></a><span data-ttu-id="41d8f-146">Substituer ReadRequestBodyAsync/WriteResponseBodyAsync</span><span class="sxs-lookup"><span data-stu-id="41d8f-146">Override ReadRequestBodyAsync/WriteResponseBodyAsync</span></span> 

<span data-ttu-id="41d8f-147">Vous effectuez le travail réel de désérialisation ou de sérialisation dans `ReadRequestBodyAsync` ou `WriteResponseBodyAsync`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-147">You do the actual work of deserializing or serializing in `ReadRequestBodyAsync` or `WriteResponseBodyAsync`.</span></span>  <span data-ttu-id="41d8f-148">Les lignes surlignées dans l’exemple suivant montrent comment obtenir des services à partir du conteneur d’injection de dépendances (vous ne pouvez pas les obtenir à partir des paramètres du constructeur).</span><span class="sxs-lookup"><span data-stu-id="41d8f-148">The highlighted lines in the following example show how to get services from the dependency injection container (you can't get them from constructor parameters).</span></span>

[!code-csharp[Main](custom-formatters/sample/Formatters/VcardOutputFormatter.cs?name=writeresponse&highlight=3-4)]

## <a name="how-to-configure-mvc-to-use-a-custom-formatter"></a><span data-ttu-id="41d8f-149">Guide pratique pour configurer MVC et utiliser un formateur personnalisé</span><span class="sxs-lookup"><span data-stu-id="41d8f-149">How to configure MVC to use a custom formatter</span></span>
 
<span data-ttu-id="41d8f-150">Pour utiliser un formateur personnalisé, ajoutez une instance de la classe du formateur à la collection `InputFormatters` ou `OutputFormatters`.</span><span class="sxs-lookup"><span data-stu-id="41d8f-150">To use a custom formatter, add an instance of the formatter class to the `InputFormatters` or `OutputFormatters` collection.</span></span>

[!code-csharp[Main](custom-formatters/sample/Startup.cs?name=mvcoptions&highlight=3-4)]

<span data-ttu-id="41d8f-151">Les formateurs sont évalués dans l’ordre dans lequel vous les insérez.</span><span class="sxs-lookup"><span data-stu-id="41d8f-151">Formatters are evaluated in the order you insert them.</span></span> <span data-ttu-id="41d8f-152">Le premier est prioritaire.</span><span class="sxs-lookup"><span data-stu-id="41d8f-152">The first one takes precedence.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="41d8f-153">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="41d8f-153">Next steps</span></span>

<span data-ttu-id="41d8f-154">Consultez l’[exemple d’application](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-formatters/sample), qui implémente de simples formateurs d’entrée et de sortie vCard.</span><span class="sxs-lookup"><span data-stu-id="41d8f-154">See the [sample application](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-formatters/sample), which implements simple vCard input and output formatters.</span></span>  <span data-ttu-id="41d8f-155">L’application lit et écrit des vCard qui ressemblent à l’exemple suivant :</span><span class="sxs-lookup"><span data-stu-id="41d8f-155">The application reads and writes vCards that look like the following example:</span></span>

```
BEGIN:VCARD
VERSION:2.1
N:Davolio;Nancy
FN:Nancy Davolio
UID:20293482-9240-4d68-b475-325df4a83728
END:VCARD
```

<span data-ttu-id="41d8f-156">Pour afficher la sortie vCard, exécutez l’application et envoyez une requête Get avec « text/vcard » dans l’en-tête Accept à `http://localhost:63313/api/contacts/` (quand l’exécution a lieu à partir de Visual Studio) ou `http://localhost:5000/api/contacts/` (quand l’exécution a lieu à partir de la ligne de commande).</span><span class="sxs-lookup"><span data-stu-id="41d8f-156">To see vCard output, run the application and send a Get request with Accept header "text/vcard" to `http://localhost:63313/api/contacts/` (when running from Visual Studio) or `http://localhost:5000/api/contacts/` (when running from the command line).</span></span>

<span data-ttu-id="41d8f-157">Pour ajouter un vCard à la collection de contacts en mémoire, envoyez une requête Post à la même URL, avec « text/vcard » dans l’en-tête Content-Type et le texte du vCard dans le corps, comme dans l’exemple ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="41d8f-157">To add a vCard to the in-memory collection of contacts, send a Post request to the same URL, with Content-Type header "text/vcard" and with vCard text in the body, formatted like the example above.</span></span>