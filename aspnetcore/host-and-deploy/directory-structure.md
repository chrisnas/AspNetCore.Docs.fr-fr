---
title: Structure de répertoires ASP.NET Core
author: guardrex
description: Découvrez la structure de répertoires des applications ASP.NET Core publiées.
monikerRange: '>= aspnetcore-2.2'
ms.author: riande
ms.custom: mvc
ms.date: 06/17/2019
uid: host-and-deploy/directory-structure
ms.openlocfilehash: f1df047decc7a0a6b7dcee57a690c55eea428b05
ms.sourcegitcommit: 28a2874765cefe9eaa068dceb989a978ba2096aa
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/17/2019
ms.locfileid: "67166977"
---
# <a name="aspnet-core-directory-structure"></a>Structure de répertoires ASP.NET Core

Par [Luke Latham](https://github.com/guardrex)

Le répertoire *publier* contient les ressources de l’application qui peuvent être déployées et produites par la commande [dotnet publish](/dotnet/core/tools/dotnet-publish). Le répertoire contient :

* Les fichiers de l’application
* Les fichiers de configuration
* Les ressources statiques
* Packages
* Un runtime ([déploiement autonome](/dotnet/core/deploying/#self-contained-deployments-scd) uniquement)

| Type d’application | Structure de répertoires |
| -------- | ------------------- |
| [Déploiement dépendant du framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd) | <ul><li>publish&dagger;<ul><li>Views&dagger; (applications MVC, si les vues ne sont pas précompilées)</li><li>Pages&dagger; (applications de pages Razor ou MVC, si les pages ne sont pas précompilées)</li><li>wwwroot&dagger;</li><li>Fichiers *\.dll</li><li>{NOM de l’ASSEMBLY}.deps.json</li><li>{NOM de l’ASSEMBLY}.dll</li><li>{NOM de l’ASSEMBLY}.pdb</li><li>{NOM de l’ASSEMBLY}.Views.dll</li><li>{NOM de l’ASSEMBLY}.Views.pdb</li><li>{NOM de l’ASSEMBLY}.runtimeconfig.json</li><li>web.config (déploiements IIS)</li></ul></li></ul> |
| [Déploiement autonome](/dotnet/core/deploying/#self-contained-deployments-scd) | <ul><li>publish&dagger;<ul><li>Views&dagger; (applications MVC, si les vues ne sont pas précompilées)</li><li>Pages&dagger; (applications de pages Razor ou MVC, si les pages ne sont pas précompilées)</li><li>wwwroot&dagger;</li><li>Fichiers \*.dll</li><li>{NOM de l’ASSEMBLY}.deps.json</li><li>{NOM de l’ASSEMBLY}.dll</li><li>{NOM de l’ASSEMBLY}.exe</li><li>{NOM de l’ASSEMBLY}.pdb</li><li>{NOM de l’ASSEMBLY}.Views.dll</li><li>{NOM de l’ASSEMBLY}.Views.pdb</li><li>{NOM de l’ASSEMBLY}.runtimeconfig.json</li><li>web.config (déploiements IIS)</li></ul></li></ul> |

&dagger;Indique un répertoire

Le répertoire *publish* représente le *chemin racine du contenu*, également appelé *chemin de base de l’application*, du déploiement. Quel que soit le nom donné au répertoire *publish* de l’application déployée sur le serveur, son emplacement sert de chemin physique, sur le serveur, de l’application hébergée.

Le répertoire *wwwroot*, s’il existe, contient uniquement des ressources statiques.

::: moniker range="< aspnetcore-3.0"

La création d’un dossier *Logs* est utile à la [journalisation de débogage améliorée du module ASP.NET Core](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs). Les dossiers situés dans le chemin fourni pour la valeur `<handlerSetting>` ne sont pas créés automatiquement par le module. Ils doivent préexister dans le déploiement pour permettre au module d’écrire dans le journal de débogage.

Vous pouvez créer le répertoire *Logs* pour le déploiement à l’aide de l’une des deux approches suivantes :

* Ajoutez l’élément `<Target>` suivant au fichier projet :

   ```xml
   <Target Name="CreateLogsFolder" AfterTargets="Publish">
     <MakeDir Directories="$(PublishDir)Logs" 
              Condition="!Exists('$(PublishDir)Logs')" />
     <WriteLinesToFile File="$(PublishDir)Logs\.log" 
                       Lines="Generated file" 
                       Overwrite="True" 
                       Condition="!Exists('$(PublishDir)Logs\.log')" />
   </Target>
   ```

   L’élément `<MakeDir>` crée un dossier *Logs* vide dans la sortie publiée. L’élément utilise la propriété `PublishDir` pour déterminer l’emplacement cible en vue de la création du dossier. Plusieurs méthodes de déploiement, telles que Web Deploy, ignorent les dossiers vides pendant le déploiement. L’élément `<WriteLinesToFile>` génère un fichier dans le dossier *Logs*, ce qui garantit le déploiement du dossier sur le serveur. La création d’un dossier à l’aide de cette approche échoue si le processus de travail n’a pas accès en écriture au dossier cible.

* Créez physiquement le répertoire *Logs* sur le serveur dans le déploiement.

Le répertoire de déploiement requiert des autorisations de lecture et d’exécution. Le répertoire *Logs* requiert des autorisations de lecture et d’écriture. D’autres répertoires où des fichiers sont écrits nécessitent des autorisations de lecture et d’écriture.

::: moniker-end

## <a name="additional-resources"></a>Ressources supplémentaires

* [dotnet publish](/dotnet/core/tools/dotnet-publish)
* [Déploiement d’applications .NET Core](/dotnet/core/deploying/)
* [Frameworks cibles](/dotnet/standard/frameworks)
* [Catalogue RID .NET Core](/dotnet/core/rid-catalog)
