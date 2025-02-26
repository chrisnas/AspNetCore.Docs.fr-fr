---
title: ASP.NET Core éblouissant de débogage
author: guardrex
description: Découvrez comment déboguer des applications éblouissantes.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: blazor/debug
ms.openlocfilehash: 9fc3f1d2dd7dc79d2ba3d64bff6e0f92ac2cf6dc
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391181"
---
# <a name="debug-aspnet-core-blazor"></a>ASP.NET Core éblouissant de débogage

[Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Une prise en charge *précoce* pour le débogage des applications webassembly éblouissantes s’exécutant sur webassembly dans Chrome.

Les fonctionnalités du débogueur sont limitées. Les scénarios disponibles sont les suivants :

* Définir et supprimer des points d’arrêt.
* Une seule étape (`F10`) à l’aide du code ou de la reprise (`F8`) de l’exécution du code.
* Dans l' *affichage des variables locales,* observez les valeurs des variables locales de type `int`, `string` et `bool`.
* Consultez la pile des appels, y compris les chaînes d’appel qui passent de JavaScript à .NET et de .NET à JavaScript.

Vous *ne pouvez pas*:

* Observez les valeurs des variables locales qui ne sont pas un `int`, `string` ou `bool`.
* Observez les valeurs de tous les champs ou propriétés de classe.
* Pointez sur les variables pour afficher leurs valeurs.
* Évaluer les expressions dans la console.
* Effectuer un pas à pas détaillé dans les appels asynchrones.
* Effectuez la plupart des autres scénarios de débogage ordinaires.

Le développement d’autres scénarios de débogage est l’un des objectifs de l’équipe d’ingénierie.

## <a name="prerequisites"></a>Configuration requise

Le débogage requiert l’un des navigateurs suivants :

* Google Chrome (version 70 ou ultérieure)
* Version préliminaire de Microsoft Edge ([canal de développement Edge](https://www.microsoftedgeinsider.com))

## <a name="procedure"></a>Procédure

1. Exécutez une application de webassembly éblouissante dans la configuration de @no__t 0. Transmettez l’option `--configuration Debug` à la commande [dotnet Run](/dotnet/core/tools/dotnet-run) : `dotnet run --configuration Debug`.
1. Accédez à l’application dans le navigateur.
1. Placez le focus clavier sur l’application, et non sur le panneau Outils de développement. Le panneau Outils de développement peut être fermé lorsque le débogage est initié.
1. Sélectionnez le raccourci clavier de éblouissant spécifique suivant :
   * `Shift+Alt+D` sur Windows/Linux
   * `Shift+Cmd+D` sur macOS
1. Suivez les étapes indiquées à l’écran pour redémarrer le navigateur en activant le débogage à distance.
1. Sélectionnez une nouvelle fois le raccourci clavier éblouissant suivant pour démarrer la session de débogage :
   * `Shift+Alt+D` sur Windows/Linux
   * `Shift+Cmd+D` sur macOS

## <a name="enable-remote-debugging"></a>Activer le débogage distant

Si le débogage distant est désactivé, une page d’erreur **Impossible de trouver un onglet de navigateur pouvant être débogué** est générée par chrome. La page d’erreur contient des instructions pour l’exécution de chrome avec le port de débogage ouvert, afin que le proxy de débogage éblouissant puisse se connecter à l’application. *Fermez toutes les instances chrome* et redémarrez Chrome comme indiqué.

## <a name="debug-the-app"></a>Déboguer l’application

Une fois que Chrome s’exécute avec le débogage distant activé, le raccourci clavier de débogage ouvre un nouvel onglet du débogueur. Après un moment, l’onglet **sources** affiche une liste des assemblys .net dans l’application. Développez chaque assembly et recherchez les fichiers sources *. cs*/ *. Razor* disponibles pour le débogage. Définissez des points d’arrêt, revenez à l’onglet de l’application, et les points d’arrêt sont atteints lorsque le code s’exécute. Après l’accès à un point d’arrêt, vous devez effectuer une seule étape (`F10`) par le biais de l’exécution du code ou de la reprise (`F8`) normalement.

Éblouissant fournit un proxy de débogage qui implémente le [protocole chrome devtools](https://chromedevtools.github.io/devtools-protocol/) et augmente le protocole avec. Informations spécifiques à .net. Quand vous appuyez sur le raccourci clavier, éblouissant pointe le DevTools chrome au niveau du proxy. Le proxy se connecte à la fenêtre du navigateur que vous cherchez à déboguer (par conséquent, il est nécessaire d’activer le débogage distant).

## <a name="browser-source-maps"></a>Mappages des sources du navigateur

Les mappages de source de navigateur permettent au navigateur de mapper les fichiers compilés à leurs fichiers sources d’origine et sont couramment utilisés pour le débogage côté client. Toutefois, éblouissant n’est actuellement pas C# mappé directement à JavaScript/WASM. À la place, éblouissant effectue une interprétation IL dans le navigateur, de sorte que les mappages de source ne sont pas pertinents.

## <a name="troubleshooting-tip"></a>Conseil de dépannage

Si vous rencontrez des erreurs, le Conseil suivant peut vous aider :

Dans l’onglet **débogueur** , ouvrez les outils de développement de votre navigateur. Dans la console, exécutez `localStorage.clear()` pour supprimer les points d’arrêt.
