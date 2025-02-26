---
title: Héberger et déployer ASP.NET Core serveur éblouissant
author: guardrex
description: Découvrez comment héberger et déployer une application de serveur éblouissante à l’aide de ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/05/2019
uid: host-and-deploy/blazor/server
ms.openlocfilehash: 693d7ff67bad3a0c5bd050b795833763056ed511
ms.sourcegitcommit: dd026eceee79e943bd6b4a37b144803b50617583
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2019
ms.locfileid: "72378812"
---
# <a name="host-and-deploy-blazor-server"></a>Héberger et déployer le serveur éblouissant

Par [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com) et [Daniel Roth](https://github.com/danroth27)

## <a name="host-configuration-values"></a>Valeurs de configuration de l’hôte

Les [applications serveur éblouissantes](xref:blazor/hosting-models#blazor-server) peuvent accepter des [valeurs de configuration d’hôte générique](xref:fundamentals/host/generic-host#host-configuration).

## <a name="deployment"></a>Déploiement

À l’aide du [modèle d’hébergement de serveur éblouissant](xref:blazor/hosting-models#blazor-server), éblouissant est exécuté sur le serveur à partir d’une application ASP.net core. Les mises à jour de l’interface utilisateur, la gestion des événements et les appels JavaScript sont gérés par le biais d’une connexion [SignalR](xref:signalr/introduction).

Un serveur web capable d’héberger une application ASP.NET Core est nécessaire. Visual Studio inclut le modèle de projet **Application serveur Blazor** (modèle `blazorserverside` quand vous utilisez la commande [dotnet new](/dotnet/core/tools/dotnet-new)).

## <a name="scalability"></a>Extensibilité

Planifiez un déploiement pour tirer le meilleur parti de l’infrastructure disponible pour une application serveur éblouissante. Consultez les ressources suivantes pour résoudre l’évolutivité de l’application serveur éblouissante :

* [Notions de base des applications serveur éblouissantes](xref:blazor/hosting-models#blazor-server)
* <xref:security/blazor/server>

### <a name="deployment-server"></a>Serveur de déploiement

Lorsque vous envisagez l’évolutivité d’un serveur unique (montée en puissance), la mémoire disponible pour une application est probablement la première ressource que l’application épuisera en fonction des demandes des utilisateurs. La mémoire disponible sur le serveur affecte les éléments suivants :

* Nombre de circuits actifs qu’un serveur peut prendre en charge.
* Latence de l’interface utilisateur sur le client.

Pour obtenir des conseils sur la création d’applications de serveur éblouissantes et sécurisées et évolutives, consultez <xref:security/blazor/server>.

Chaque circuit utilise environ 250 Ko de mémoire pour une application de type *Hello World*minimale. La taille d’un circuit dépend du code de l’application et des exigences de maintenance d’état associées à chaque composant. Nous vous recommandons de mesurer les demandes de ressources pendant le développement de votre application et de votre infrastructure, mais la ligne de base suivante peut être un point de départ pour la planification de votre cible de déploiement : Si vous vous attendez à ce que votre application prenne en charge 5 000 utilisateurs simultanés, pensez à la budgétisation à moins de 1,3 Go de mémoire serveur pour l’application (ou ~ 273 Ko par utilisateur).

### <a name="signalr-configuration"></a>Configuration SignalR

Les applications serveur éblouissantes utilisent ASP.NET Core Signalr pour communiquer avec le navigateur. [Les conditions d’hébergement et de mise à l’échelle de signalr](xref:signalr/publish-to-azure-web-app) s’appliquent aux applications de serveur éblouissantes.

Éblouissant fonctionne mieux lorsque vous utilisez WebSockets comme transport Signalr en raison d’une latence, d’une fiabilité et d’une [sécurité](xref:signalr/security)moindres. L’interrogation longue est utilisée par Signalr lorsque WebSocket n’est pas disponible ou lorsque l’application est configurée de manière explicite pour utiliser une interrogation longue. Lors du déploiement sur Azure App Service, configurez l’application pour qu’elle utilise WebSockets dans les paramètres Portail Azure pour le service. Pour plus d’informations sur la configuration de l’application pour Azure App Service, consultez les [instructions de publication signalr](xref:signalr/publish-to-azure-web-app).

Nous vous recommandons d’utiliser le [service Azure signalr](/azure/azure-signalr) pour les applications serveur éblouissantes. Le service permet de mettre à l’échelle une application de serveur éblouissant sur un grand nombre de connexions Signalr simultanées. En outre, la portée mondiale et les centres de données haute performance du service Signalr contribuent de manière significative à réduire la latence en raison de la géographie. Pour configurer une application (et éventuellement approvisionner) le service Azure Signalr :

* Créez un profil de publication Azure Apps dans Visual Studio pour l’application de serveur éblouissant.
* Ajoutez la dépendance du **service Azure signalr** au profil. Si l’abonnement Azure ne dispose pas d’une instance de service Azure Signalr préexistante à attribuer à l’application, sélectionnez **créer une instance de service Azure signalr** pour approvisionner une nouvelle instance de service.
* Publiez l’application sur Azure.

### <a name="measure-network-latency"></a>Mesurer la latence du réseau

L' [interopérabilité js](xref:blazor/javascript-interop) peut être utilisée pour mesurer la latence du réseau, comme le montre l’exemple suivant :

```cshtml
@inject IJSRuntime JS

@if (latency is null)
{
    <span>Calculating...</span>
}
else
{
    <span>@(latency.Value.TotalMilliseconds)ms</span>
}

@code
{
    private DateTime startTime;
    private TimeSpan? latency;

    protected override async Task OnInitializedAsync()
    {
        startTime = DateTime.UtcNow;
        var _ = await JS.InvokeAsync<string>("toString");
        latency = DateTime.UtcNow - startTime;
    }
}
```

Pour une expérience d’interface utilisateur raisonnable, nous recommandons une latence d’interface utilisateur soutenue de 250 ms ou moins.
