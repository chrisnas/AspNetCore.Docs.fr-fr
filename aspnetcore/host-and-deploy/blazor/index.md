---
title: Héberger et déployer ASP.NET Core Blazor
author: guardrex
description: Découvrez comment héberger et déployer des applications Blazor.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: host-and-deploy/blazor/index
ms.openlocfilehash: 271135a0ebe67d31fd8e2bcf672e723814727147
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391336"
---
# <a name="host-and-deploy-aspnet-core-blazor"></a>Héberger et déployer ASP.NET Core Blazor

Par [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com) et [Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

## <a name="publish-the-app"></a>Publier l'application

Les applications sont publiées pour le déploiement dans la configuration Release.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Sélectionnez **Build** > **Publier {APPLICATION}** dans la barre de navigation.
1. Sélectionnez l’onglet *Cible de publication*. Pour publier localement, sélectionnez **Dossier**.
1. Acceptez l’emplacement par défaut dans le champ **Choisir un dossier** ou spécifiez un autre emplacement. Sélectionnez le bouton **Publier**.

# <a name="net-core-clitabnetcore-cli"></a>[CLI .NET Core](#tab/netcore-cli)

Utilisez la commande [dotnet publish](/dotnet/core/tools/dotnet-publish) pour publier l’application avec une configuration Release :

```dotnetcli
dotnet publish -c Release
```

---

La publication de l’application déclenche une [restauration](/dotnet/core/tools/dotnet-restore) des dépendances du projet et [crée](/dotnet/core/tools/dotnet-build) le projet avant de créer les ressources pour le déploiement. Dans le cadre du processus de génération, les assemblys et méthodes inutilisés sont supprimés pour réduire la durée du chargement et la taille du téléchargement de l’application.

Une application de webassembly éblouissant est publiée dans le dossier */bin/Release/{Target Framework}/Publish/{assembly/dist* . Une application de serveur éblouissant est publiée dans le dossier */bin/Release/{Target Framework}/Publish* .

Les ressources du dossier sont déployées sur le serveur web. Le déploiement peut être un processus manuel ou automatisé, en fonction des outils de développement utilisés.

## <a name="app-base-path"></a>Chemin de base de l’application

Le *chemin d’accès de base* de l’application est le chemin de l’URL racine de l’application. Prenons l’exemple de l’application principale et de l’application éblouissant suivante :

* L’application principale est appelée `MyApp` :
  * L’application se trouve physiquement à l’adresse *d : \\MyApp*.
  * Les demandes sont reçues à `https://www.contoso.com/{MYAPP RESOURCE}`.
* Une application éblouissante appelée `CoolApp` est une sous-application de `MyApp` :
  * La sous-application réside physiquement dans *d : \\MyApp @ no__t-2CoolApp*.
  * Les demandes sont reçues à `https://www.contoso.com/CoolApp/{COOLAPP RESOURCE}`.

Si vous ne spécifiez pas de configuration supplémentaire pour `CoolApp`, la sous-application dans ce scénario n’a aucune connaissance de son emplacement sur le serveur. Par exemple, l’application ne peut pas construire des URL relatives correctes pour ses ressources sans savoir qu’elle réside au chemin d’URL relatif `/CoolApp/`.

Pour fournir la configuration du chemin d’accès de base de l’application éblouissante `https://www.contoso.com/CoolApp/`, l’attribut `href` de la balise @no__t 1 est défini sur le chemin d’accès racine relatif dans le fichier *wwwroot/index.html* :

```html
<base href="/CoolApp/">
```

En fournissant le chemin d’URL relatif, un composant qui ne se trouve pas dans le répertoire racine peut construire des URL relatives au chemin d’accès racine de l’application. Les composants à différents niveaux de la structure de répertoires peuvent créer des liens vers d’autres ressources à des emplacements de l’application. Le chemin de base de l’application sert également à intercepter les clics sur les liens hypertextes où la cible `href` du lien se trouve dans l’espace d’URI du chemin de base de l’application (le routeur Blazor gère la navigation interne).

Dans de nombreux scénarios d’hébergement, le chemin d’URL relatif à l’application est la racine de l’application. Dans ce cas, le chemin d’accès de base de l’URL relative de l’application est une barre oblique (`<base href="/" />`), qui est la configuration par défaut pour une application éblouissante. Dans d’autres scénarios d’hébergement, tels que les pages GitHub et les sous-applications IIS, le chemin d’accès de base de l’application doit être défini sur le chemin d’URL relatif du serveur vers l’application.

Pour définir le chemin de base de l’application, mettez à jour la balise `<base>` dans les éléments de balise `<head>` du fichier *wwwroot/index.html*. Définissez la valeur de l’attribut `href` sur `/{RELATIVE URL PATH}/` (la barre oblique de fin est requise), où `{RELATIVE URL PATH}` est le chemin d’URL relatif complet de l’application.

Pour une application avec un chemin d’URL relatif non racine (par exemple, `<base href="/CoolApp/">`), l’application ne parvient pas à trouver ses ressources lorsqu’elle est *exécutée localement*. Pour surmonter ce problème pendant le développement local et les tests, vous pouvez fournir un argument de *base de chemin* qui correspond à la valeur `href` de la balise `<base>` au moment de l’exécution. Pour passer l’argument de base Path lors de l’exécution locale de l’application, exécutez la commande `dotnet run` à partir du répertoire de l’application avec l’option `--pathbase` :

```dotnetcli
dotnet run --pathbase=/{RELATIVE URL PATH (no trailing slash)}
```

Pour une application avec un chemin d’URL relatif de `/CoolApp/` (`<base href="/CoolApp/">`), la commande est :

```dotnetcli
dotnet run --pathbase=/CoolApp
```

L’application répond localement à `http://localhost:port/CoolApp`.

## <a name="deployment"></a>Déploiement

Pour obtenir des conseils de déploiement, consultez les rubriques suivantes :

* <xref:host-and-deploy/blazor/webassembly>
* <xref:host-and-deploy/blazor/server>
