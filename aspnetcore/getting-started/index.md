---
title: Bien démarrer avec ASP.NET Core
author: rick-anderson
description: Bref tutoriel qui crée et exécute une application Hello World de base à l’aide d’ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 09/22/2019
uid: getting-started
ms.openlocfilehash: 116a22bce80257948bfcc02c03a74a4b5568b8b5
ms.sourcegitcommit: 4649814d1ae32248419da4e8f8242850fd8679a5
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2019
ms.locfileid: "71975692"
---
# <a name="tutorial-get-started-with-aspnet-core"></a>Tutoriel : Bien démarrer avec ASP.NET Core

Ce tutoriel montre comment utiliser l’interface de ligne de commande .NET Core pour créer et exécuter une application web ASP.NET Core.

Vous allez apprendre à :

> [!div class="checklist"]
> * Créer un projet application web.
> * Approuver le certificat de développement.
> * Exécuter l’application.
> * Modifier une page Razor.

À la fin du tutoriel, vous disposerez d’une application web qui fonctionne et s’exécute sur votre machine locale.

![Page d’accueil d’application web](_static/home-page.png)

## <a name="prerequisites"></a>Prérequis

[!INCLUDE[](~/includes/3.0-SDK.md)]

## <a name="create-a-web-app-project"></a>Créer un projet application web

Ouvrez un interpréteur de commandes, puis entrez la commande suivante :

```dotnetcli
dotnet new webapp -o aspnetcoreapp
```

La commande précédente :

* Crée une application Web.  
* Le paramètre `-o aspnetcoreapp` crée un répertoire nommé *aspnetcoreapp* avec les fichiers sources de l’application.

### <a name="trust-the-development-certificate"></a>Approuver le certificat de développement

Approuvez le certificat de développement HTTPS :

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

```dotnetcli
dotnet dev-certs https --trust
```

La commande précédente affiche la boîte de dialogue suivante :

![Boîte de dialogue Avertissement de sécurité](~/getting-started/_static/cert.png)

Sélectionnez **Oui** si vous acceptez d’approuver le certificat de développement.

# <a name="macostabmacos"></a>[macOS](#tab/macos)

```dotnetcli
dotnet dev-certs https --trust
```

La commande précédente affiche le message suivant :

*L’approbation du certificat de développement HTTPS a été demandée. Si le certificat n’a pas encore été approuvé, nous exécutons la commande suivante :* `'sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain <<certificate>>'`

Cette commande risque de vous demander votre mot de passe afin d’installer le certificat sur le trousseau système. Entrez votre mot de passe si vous acceptez d’approuver le certificat de développement.

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

Consultez la documentation de votre distribution Linux pour savoir comment approuver le certificat de développement HTTPS.

---

Pour plus d’informations, consultez [Approuver le certificat de développement ASP.NET Core HTTPS](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)

## <a name="run-the-app"></a>Exécuter l'application

Exécutez les commandes suivantes :

```dotnetcli
cd aspnetcoreapp
dotnet watch run
```

Une fois que l’interface de commande indique que l’application a démarré, accédez à [https://localhost:5001](https://localhost:5001).

## <a name="edit-a-razor-page"></a>Modifier une page Razor

Ouvrez *pages/index. cshtml* et modifiez et enregistrez la page avec le balisage en surbrillance suivant :

[!code-cshtml[](sample/index.cshtml?highlight=9)]

Accédez à [https://localhost:5001](https://localhost:5001), actualisez la page et vérifiez que les modifications sont affichées.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Créer un projet application web.
> * Approuver le certificat de développement.
> * Exécuter le projet.
> * Effectuer une modification.

Pour en savoir plus sur ASP.NET Core, consultez le parcours d’apprentissage recommandé dans la présentation :

> [!div class="nextstepaction"]
> <xref:index#recommended-learning-path>
