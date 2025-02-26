---
title: Héberger et déployer ASP.NET Core
author: guardrex
description: Découvrez comment configurer des environnements d’hébergement et déployer des applications ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/12/2019
uid: host-and-deploy/index
ms.openlocfilehash: 068c5f357b8010d78493a78a10c353e3317f3867
ms.sourcegitcommit: b4ef2b00f3e1eb287138f8b43c811cb35a100d3e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/21/2019
ms.locfileid: "65969884"
---
# <a name="host-and-deploy-aspnet-core"></a>Héberger et déployer ASP.NET Core

En général, pour déployer une application ASP.NET Core sur un environnement d’hébergement :

* Déployer l’application publiée dans un dossier sur le serveur d’hébergement.
* Configurer un gestionnaire de processus qui démarre l’application à l’arrivée des requêtes et redémarre l’application en cas de blocage ou quand le serveur redémarre.
* Pour la configuration d’un proxy inverse, configurez un proxy inverse pour transférer les requêtes à l’application.

## <a name="publish-to-a-folder"></a>Publier dans un dossier

La commande [dotnet publish](/dotnet/core/tools/dotnet-publish) compile le code d’application et copie les fichiers nécessaires pour exécuter l’application dans un dossier *publish*. Dans le cadre d’un déploiement à partir de Visual Studio, l’étape `dotnet publish` est effectuée automatiquement avant que les fichiers ne soient copiés vers la destination du déploiement.

### <a name="folder-contents"></a>Contenu du dossier

Le dossier *publish* contient un ou plusieurs fichiers d’assembly d’application, ses dépendances et éventuellement le runtime .NET.

Une application .NET Core peut être publiée en tant que *déploiement autonome* ou *déploiement dépendant du framework*. Si l’application est autonome, les fichiers d’assembly qui contiennent le runtime .NET sont inclus dans le dossier *publish*. Si l’application dépend du framework, les fichiers du runtime .NET ne sont pas inclus, car l’application a une référence à une version de .NET installée sur le serveur. Le modèle de déploiement par défaut est « dépendante du framework ». Pour plus d’informations, consultez [Déploiement d’applications .NET Core](/dotnet/core/deploying/).

En plus des fichiers *.exe* et *.dll*, le dossier *publish* d’une application ASP.NET Core contient généralement des fichiers de configuration, des ressources statiques et des vues MVC. Pour plus d'informations, consultez <xref:host-and-deploy/directory-structure>.

## <a name="set-up-a-process-manager"></a>Configurer un gestionnaire de processus

Une application ASP.NET Core est une application console qui doit être démarrée quand un serveur démarre, et redémarrée si elle se bloque. Pour automatiser le démarrage et le redémarrage, un gestionnaire de processus est nécessaire. Les gestionnaires de processus les plus courants pour ASP.NET Core sont les suivants :

* Linux
  * [Nginx](xref:host-and-deploy/linux-nginx)
  * [Apache](xref:host-and-deploy/linux-apache)
* Windows
  * [IIS](xref:host-and-deploy/iis/index)
  * [Service Windows](xref:host-and-deploy/windows-service)

## <a name="set-up-a-reverse-proxy"></a>Configurer un proxy inverse

Si l’application utilise le serveur [Kestrel](xref:fundamentals/servers/kestrel), vous pouvez utiliser [Nginx](xref:host-and-deploy/linux-nginx), [Apache](xref:host-and-deploy/linux-apache) ou [IIS](xref:host-and-deploy/iis/index) comme serveur proxy inverse. Un serveur proxy inverse reçoit les requêtes HTTP en provenance d’Internet et les transmet à Kestrel.

Les deux configurations, &mdash;avec ou sans serveur proxy inverse&mdash;, sont des configurations d’hébergement prises en charge. Pour plus d’informations, consultez [Quand utiliser Kestrel avec un proxy inverse ?](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy).

## <a name="proxy-server-and-load-balancer-scenarios"></a>Scénarios avec un serveur proxy et un équilibreur de charge

Une configuration supplémentaire peut être nécessaire pour les applications hébergées derrière des serveurs proxy et des équilibreurs de charge. Sans configuration supplémentaire, une application peut ne pas avoir accès au schéma (HTTP/HTTPS) et à l’adresse IP distante d’où provient une requête. Pour plus d’informations, consultez [Configurer ASP.NET Core pour l’utilisation de serveurs proxy et d’équilibreurs de charge](xref:host-and-deploy/proxy-load-balancer).

## <a name="use-visual-studio-and-msbuild-to-automate-deployments"></a>Utiliser Visual Studio et MSBuild pour automatiser les déploiements

Le déploiement nécessite souvent des tâches supplémentaires en plus de la copie de la sortie de [dotnet publish](/dotnet/core/tools/dotnet-publish) vers un serveur. Par exemple, des fichiers supplémentaires peuvent être requis ou exclus du dossier *publish*. Visual Studio utilise MSBuild pour le déploiement web, et MSBuild peut être personnalisé pour effectuer de nombreuses autres tâches pendant le déploiement. Pour plus d’informations, consultez <xref:host-and-deploy/visual-studio-publish-profiles> et l’ouvrage [Using MSBuild and Team Foundation Build](http://msbuildbook.com/) (Utilisation de MSBuild et Team Foundation Build).

Les applications peuvent être déployées directement à partir de Visual Studio sur Azure App Service à l’aide de la [fonctionnalité de publication web](xref:tutorials/publish-to-azure-webapp-using-vs) ou de la [prise en charge intégrée de Git](xref:host-and-deploy/azure-apps/azure-continuous-deployment). Azure DevOps Services prend en charge le [déploiement continu sur Azure App Service](/azure/devops/pipelines/targets/webapp). Pour plus d’informations, consultez [DevOps avec ASP.NET Core et Azure](xref:azure/devops/index).

## <a name="publish-to-azure"></a>Publier sur Azure

Consultez <xref:tutorials/publish-to-azure-webapp-using-vs> pour obtenir des instructions sur la façon de publier une application sur Azure à l’aide de Visual Studio. Un exemple supplémentaire est fourni par [Créer une application web ASP.NET Core dans Azure](/azure/app-service/app-service-web-get-started-dotnet).

## <a name="publish-with-msdeploy-on-windows"></a>Publier avec MSDeploy sur Windows

Consultez <xref:host-and-deploy/visual-studio-publish-profiles> pour obtenir des instructions sur la publication d’une application avec un profil de publication Visual Studio, notamment à partir d’une invite de commandes Windows à l’aide de la commande [dotnet msbuild](/dotnet/core/tools/dotnet-msbuild).

## <a name="host-in-a-web-farm"></a>Héberger dans une batterie de serveurs web

Pour plus d’informations sur la configuration pour héberger des applications ASP.NET Core dans un environnement de batterie de serveurs web (par exemple, le déploiement de plusieurs instances de votre application pour la scalabilité), consultez <xref:host-and-deploy/web-farm>.

::: moniker range=">= aspnetcore-2.2"

## <a name="perform-health-checks"></a>Effectuer des contrôles d’intégrité

Utilisez Health Check Middleware pour effectuer des contrôles d’intégrité sur une application et ses dépendances. Pour plus d'informations, consultez <xref:host-and-deploy/health-checks>.

::: moniker-end

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:host-and-deploy/docker/index>
* <xref:test/troubleshoot>
