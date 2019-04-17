---
title: Migration des services de gRPC à partir de C vers ASP.NET Core
author: juntaoluo
description: Découvrez comment déplacer une application de gRPC en C-core existante s’exécutant sur la pile ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 03/31/2019
uid: grpc/migration
ms.openlocfilehash: ffe5ccbd99c6920e093eddc00fc60a9f66aab527
ms.sourcegitcommit: 5995f44e9e13d7e7aa8d193e2825381c42184e47
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/02/2019
ms.locfileid: "59515511"
---
# <a name="migrating-grpc-services-from-c-core-to-aspnet-core"></a><span data-ttu-id="b2f39-103">Migration des services de gRPC à partir de C vers ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="b2f39-103">Migrating gRPC services from C-core to ASP.NET Core</span></span>

<span data-ttu-id="b2f39-104">Par [John Luo](https://github.com/juntaoluo)</span><span class="sxs-lookup"><span data-stu-id="b2f39-104">By [John Luo](https://github.com/juntaoluo)</span></span>

<span data-ttu-id="b2f39-105">En raison de l’implémentation de la pile sous-jacente, toutes les fonctionnalités ne fonctionnent de la même façon entre [gRPC par cœur-C](https://grpc.io/blog/grpc-stacks) applications et les applications ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="b2f39-105">Due to the implementation of the underlying stack, not all features work in the same way between [C-core-based gRPC](https://grpc.io/blog/grpc-stacks) apps and ASP.NET Core-based apps.</span></span> <span data-ttu-id="b2f39-106">Ce document met en évidence les principales différences pour la migration entre les deux piles.</span><span class="sxs-lookup"><span data-stu-id="b2f39-106">This document highlights the key differences for migrating between the two stacks.</span></span>

## <a name="grpc-service-implementation-lifetime"></a><span data-ttu-id="b2f39-107">durée de vie de gRPC service implémentation</span><span class="sxs-lookup"><span data-stu-id="b2f39-107">gRPC service implementation lifetime</span></span>

<span data-ttu-id="b2f39-108">Dans la pile ASP.NET Core, services gRPC, par défaut, sont créés avec un [durée de vie délimitée](xref:fundamentals/dependency-injection#service-lifetimes).</span><span class="sxs-lookup"><span data-stu-id="b2f39-108">In the ASP.NET Core stack, gRPC services, by default, are created with a [scoped lifetime](xref:fundamentals/dependency-injection#service-lifetimes).</span></span> <span data-ttu-id="b2f39-109">En revanche, gRPC C-core par défaut liée à un service avec un [durée de vie singleton](xref:fundamentals/dependency-injection#service-lifetimes).</span><span class="sxs-lookup"><span data-stu-id="b2f39-109">In contrast, gRPC C-core by default binds to a service with a [singleton lifetime](xref:fundamentals/dependency-injection#service-lifetimes).</span></span>

<span data-ttu-id="b2f39-110">Une durée de vie délimitée permet à l’implémentation de service résoudre les autres services avec des durées de vie délimitées.</span><span class="sxs-lookup"><span data-stu-id="b2f39-110">A scoped lifetime allows the service implementation to resolve other services with scoped lifetimes.</span></span> <span data-ttu-id="b2f39-111">Par exemple, une durée de vie délimitée peut également résoudre `DBContext` à partir du conteneur d’injection de dépendances via l’injection de constructeur.</span><span class="sxs-lookup"><span data-stu-id="b2f39-111">For example, a scoped lifetime can also resolve `DBContext` from the DI container through constructor injection.</span></span> <span data-ttu-id="b2f39-112">À l’aide de la durée de vie délimitée :</span><span class="sxs-lookup"><span data-stu-id="b2f39-112">Using scoped lifetime:</span></span>

* <span data-ttu-id="b2f39-113">Une nouvelle instance de l’implémentation du service est construite pour chaque requête.</span><span class="sxs-lookup"><span data-stu-id="b2f39-113">A new instance of the service implementation is constructed for each request.</span></span>
* <span data-ttu-id="b2f39-114">Il n’est pas possible de partager l’état entre les demandes via les membres d’instance sur le type d’implémentation.</span><span class="sxs-lookup"><span data-stu-id="b2f39-114">It isn't possible to share state between requests via instance members on the implementation type.</span></span>
* <span data-ttu-id="b2f39-115">L’attente consiste à stocker des états partagés dans un service singleton dans le conteneur d’injection de dépendances.</span><span class="sxs-lookup"><span data-stu-id="b2f39-115">The expectation is to store shared states in a singleton service in the DI container.</span></span> <span data-ttu-id="b2f39-116">Les états partagés stockées sont résolus dans le constructeur de l’implémentation du service gRPC.</span><span class="sxs-lookup"><span data-stu-id="b2f39-116">The stored shared states are resolved in the constructor of the gRPC service implementation.</span></span> 

<span data-ttu-id="b2f39-117">Pour plus d’informations sur les durées de vie de service, consultez <xref:fundamentals/dependency-injection#service-lifetimes>.</span><span class="sxs-lookup"><span data-stu-id="b2f39-117">For more information on service lifetimes, see <xref:fundamentals/dependency-injection#service-lifetimes>.</span></span>

### <a name="add-a-singleton-service"></a><span data-ttu-id="b2f39-118">Ajouter un service singleton</span><span class="sxs-lookup"><span data-stu-id="b2f39-118">Add a singleton service</span></span>

<span data-ttu-id="b2f39-119">Pour faciliter la transition à partir d’une implémentation de C-core gRPC vers ASP.NET Core, il est possible de modifier la durée de vie du service de l’implémentation du service en limitées à un singleton.</span><span class="sxs-lookup"><span data-stu-id="b2f39-119">To facilitate the transition from a gRPC C-core implementation to ASP.NET Core, it's possible to change the service lifetime of the service implementation from scoped to singleton.</span></span> <span data-ttu-id="b2f39-120">Cela implique l’ajout d’une instance de l’implémentation du service dans le conteneur d’injection de dépendance :</span><span class="sxs-lookup"><span data-stu-id="b2f39-120">This involves adding an instance of the service implementation to the DI container:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc();
    services.AddSingleton(new GreeterService());
}
```

<span data-ttu-id="b2f39-121">Toutefois, une implémentation de service avec une durée de vie singleton n’est plus en mesure de résoudre des services délimités par le biais d’injection de constructeur.</span><span class="sxs-lookup"><span data-stu-id="b2f39-121">However, a service implementation with a singleton lifetime is no longer able to resolve scoped services through constructor injection.</span></span>

## <a name="configure-grpc-services-options"></a><span data-ttu-id="b2f39-122">Configurer les options de services gRPC</span><span class="sxs-lookup"><span data-stu-id="b2f39-122">Configure gRPC services options</span></span>

<span data-ttu-id="b2f39-123">Dans C-core-applications, paramètres tels que `grpc.max_receive_message_length` et `grpc.max_send_message_length` sont configurés avec `ChannelOption` lorsque [construction de l’instance de serveur](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server__ctor_System_Collections_Generic_IEnumerable_Grpc_Core_ChannelOption__).</span><span class="sxs-lookup"><span data-stu-id="b2f39-123">In C-core-based apps, settings such as `grpc.max_receive_message_length` and `grpc.max_send_message_length` are configured with `ChannelOption` when [constructing the Server instance](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server__ctor_System_Collections_Generic_IEnumerable_Grpc_Core_ChannelOption__).</span></span>

<span data-ttu-id="b2f39-124">Dans ASP.NET Core, `GrpcServiceOptions` permet de configurer ces paramètres.</span><span class="sxs-lookup"><span data-stu-id="b2f39-124">In ASP.NET Core, `GrpcServiceOptions` provides a way to configure these settings.</span></span> <span data-ttu-id="b2f39-125">Les paramètres peuvent être appliquées globalement à tous les services de gRPC ou à un type d’implémentation de service individuels.</span><span class="sxs-lookup"><span data-stu-id="b2f39-125">The settings can be applied globally to all gRPC services or to an individual service implementation type.</span></span> <span data-ttu-id="b2f39-126">Options spécifiées pour les types d’implémentation de service individuels remplacent les paramètres globaux du configuré.</span><span class="sxs-lookup"><span data-stu-id="b2f39-126">Options specified for individual service implementation types override global settings when configured.</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services
        .AddGrpc(globalOptions =>
        {
            // Global settings
            globalOptions.SendMaxMessageSize = 4096
            globalOptions.ReceiveMaxMessageSize = 4096
        })
        .AddServiceOptions<GreeterService>(greeterOptions =>
        {
            // GreeterService settings. These will override global settings
            globalOptions.SendMaxMessageSize = 2048
            globalOptions.ReceiveMaxMessageSize = 2048
        })
}
```

## <a name="logging"></a><span data-ttu-id="b2f39-127">Journalisation</span><span class="sxs-lookup"><span data-stu-id="b2f39-127">Logging</span></span>

<span data-ttu-id="b2f39-128">C-core-applications s’appuient sur le `GrpcEnvironment` à [configurer l’enregistreur d’événements](https://grpc.io/grpc/csharp/api/Grpc.Core.GrpcEnvironment.html?q=size#Grpc_Core_GrpcEnvironment_SetLogger_Grpc_Core_Logging_ILogger_) à des fins de débogage.</span><span class="sxs-lookup"><span data-stu-id="b2f39-128">C-core-based apps rely on the `GrpcEnvironment` to [configure the logger](https://grpc.io/grpc/csharp/api/Grpc.Core.GrpcEnvironment.html?q=size#Grpc_Core_GrpcEnvironment_SetLogger_Grpc_Core_Logging_ILogger_) for debugging purposes.</span></span> <span data-ttu-id="b2f39-129">La pile ASP.NET Core fournit cette fonctionnalité via le [API de journalisation](xref:fundamentals/logging/index).</span><span class="sxs-lookup"><span data-stu-id="b2f39-129">The ASP.NET Core stack provides this functionality through the [Logging API](xref:fundamentals/logging/index).</span></span> <span data-ttu-id="b2f39-130">Par exemple, un enregistreur d’événements peut être ajoutée au service gRPC via l’injection de constructeur :</span><span class="sxs-lookup"><span data-stu-id="b2f39-130">For example, a logger can be added to the gRPC service via constructor injection:</span></span>

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

## <a name="https"></a><span data-ttu-id="b2f39-131">HTTPS</span><span class="sxs-lookup"><span data-stu-id="b2f39-131">HTTPS</span></span>

<span data-ttu-id="b2f39-132">C-core-applications configurer HTTPS via le [Server.Ports propriété](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server_Ports).</span><span class="sxs-lookup"><span data-stu-id="b2f39-132">C-core-based apps configure HTTPS through the [Server.Ports property](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server_Ports).</span></span> <span data-ttu-id="b2f39-133">Un concept similaire est utilisé pour configurer des serveurs dans ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="b2f39-133">A similar concept is used to configure servers in ASP.NET Core.</span></span> <span data-ttu-id="b2f39-134">Par exemple, Kestrel utilise [configuration de point de terminaison](xref:fundamentals/servers/kestrel#endpoint-configuration) pour cette fonctionnalité.</span><span class="sxs-lookup"><span data-stu-id="b2f39-134">For example, Kestrel uses [endpoint configuration](xref:fundamentals/servers/kestrel#endpoint-configuration) for this functionality.</span></span>

## <a name="interceptors-and-middleware"></a><span data-ttu-id="b2f39-135">Intergiciel (middleware) et des intercepteurs</span><span class="sxs-lookup"><span data-stu-id="b2f39-135">Interceptors and Middleware</span></span>

<span data-ttu-id="b2f39-136">ASP.NET Core [intergiciel (middleware)](xref:fundamentals/middleware/index) offre des fonctionnalités similaires par rapport aux intercepteurs dans les applications gRPC par cœur-C.</span><span class="sxs-lookup"><span data-stu-id="b2f39-136">ASP.NET Core [middleware](xref:fundamentals/middleware/index) offers similar functionalities compared to interceptors in C-core-based gRPC apps.</span></span> <span data-ttu-id="b2f39-137">Intergiciel (middleware) et les intercepteurs sont conceptuellement les mêmes que les deux sont utilisés pour construire un pipeline qui gère une demande de gRPC.</span><span class="sxs-lookup"><span data-stu-id="b2f39-137">Middleware and interceptors are conceptually the same as both are used to construct a pipeline that handles a gRPC request.</span></span> <span data-ttu-id="b2f39-138">Les deux permettent de travail à exécuter avant ou après le composant suivant dans le pipeline.</span><span class="sxs-lookup"><span data-stu-id="b2f39-138">They both allow work to be performed before or after the next component in the pipeline.</span></span> <span data-ttu-id="b2f39-139">Toutefois, intergiciel (middleware) ASP.NET Core fonctionne sur les messages HTTP/2 sous-jacents, tandis qu’intercepteurs fonctionnent sur la couche d’abstraction à l’aide de gRPC le [ServerCallContext](https://grpc.io/grpc/csharp/api/Grpc.Core.ServerCallContext.html).</span><span class="sxs-lookup"><span data-stu-id="b2f39-139">However, ASP.NET Core middleware operates on the underlying HTTP/2 messages, while interceptors operate on the gRPC layer of abstraction using the [ServerCallContext](https://grpc.io/grpc/csharp/api/Grpc.Core.ServerCallContext.html).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="b2f39-140">Ressources supplémentaires</span><span class="sxs-lookup"><span data-stu-id="b2f39-140">Additional resources</span></span>

* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/aspnetcore>
* <xref:tutorials/grpc/grpc-start>