---
title: Authentification à l’aide de fournisseurs externes (Facebook, Google et autres) dans ASP.NET Core
author: rick-anderson
description: Ce didacticiel montre comment générer une application ASP.NET Core 2.x à l’aide d’OAuth 2.0 avec des fournisseurs d’authentification externes.
ms.author: riande
ms.custom: mvc
ms.date: 05/10/2019
uid: security/authentication/social/index
ms.openlocfilehash: edaf9eeaf02879b2f7816bab0eb373a7de640c05
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/18/2019
ms.locfileid: "71082504"
---
# <a name="facebook-google-and-external-provider-authentication-in-aspnet-core"></a>Authentification à l’aide de fournisseurs externes (Facebook, Google et autres) dans ASP.NET Core

Par [Valeriy Novytskyy](https://github.com/01binary) et [Rick Anderson](https://twitter.com/RickAndMSFT)

Ce tutoriel montre comment créer une application ASP.NET Core 2.2 qui permet aux utilisateurs de se connecter à l’aide d’OAuth 2.0 avec des informations d’identification provenant de fournisseurs d’authentification externes.

Les fournisseurs [Facebook](xref:security/authentication/facebook-logins), [Twitter](xref:security/authentication/twitter-logins), [Google](xref:security/authentication/google-logins) et [Microsoft](xref:security/authentication/microsoft-logins) sont traités dans les sections qui suivent. D’autres fournisseurs sont disponibles dans des packages tiers, comme [AspNet.Security.OAuth.Providers](https://github.com/aspnet-contrib/AspNet.Security.OAuth.Providers) et [AspNet.Security.OpenId.Providers](https://github.com/aspnet-contrib/AspNet.Security.OpenId.Providers).

![Icônes de réseau social pour Facebook, Twitter, Google plus et Windows](index/_static/social.png)

Permettre aux utilisateurs de se connecter avec leurs informations d’identification existantes :
* Est pratique pour les utilisateurs.
* Transfère une grande partie des complexités de la gestion du processus de connexion à un tiers. 

Pour obtenir des exemples de la façon dont les connexions des réseaux sociaux peuvent amener du trafic et des conversions de clients, consultez les études de cas par [Facebook](https://www.facebook.com/unsupportedbrowser) et [Twitter](https://dev.twitter.com/resources/case-studies).

## <a name="create-a-new-aspnet-core-project"></a>Créer un projet ASP.NET Core

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Créer un nouveau projet.
* Sélectionnez **Nouvelle application web ASP.NET Core** et **Suivant**.
* Fournissez un **Nom de projet** et confirmez ou changez l’**Emplacement**. Sélectionnez **Créer**.
* Sélectionnez **ASP.NET Core 2.2** dans la liste déroulante. Sélectionnez **Application web** dans la liste des modèles.
* Sous **Authentification**, sélectionnez **Changer** et définissez l’authentification sur **Comptes d’utilisateur individuels**. Sélectionnez **OK**.
* Dans la fenêtre **Créer une application web ASP.NET Core**, sélectionnez **Créer**.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Ouvrez le [terminal intégré](https://code.visualstudio.com/docs/editor/integrated-terminal).

* Accédez à un répertoire (`cd`) destiné à contenir le projet.

* Exécutez les commandes suivantes :

  ```dotnetcli
  dotnet new webapp -o WebApp1 -au Individual -uld
  code -r WebApp1
  ```

  * La commande `dotnet new` crée un nouveau projet Razor Pages dans le dossier *WebApp1*.
  * `-uld` utilise la Base de données locale au lieu de SQLite. Omettez `-uld` pour utiliser SQLite.
  * `-au Individual` crée le code servant à l’authentification individuelle.
  * La commande `code` ouvre le dossier *WebApp1* dans une nouvelle instance de Visual Studio Code.

* Une boîte de dialogue apparaît et affiche **Les composants nécessaires à la build et au débogage sont manquants dans « WebApp1 ». Faut-il les ajouter ?** Sélectionnez **Oui**.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

* Sélectionnez **Fichier** > **Nouvelle solution**.
* Sélectionnez **.NET Core** > **Application** dans la barre latérale. Sélectionnez le modèle **Application web**. Sélectionnez **Suivant**.
* Définissez la liste déroulante **Framework cible** sur **.NET Core 2.2**. Sélectionnez **Suivant**.
* Fournissez un **Nom de projet**. Confirmez ou changez l’**Emplacement**. Sélectionnez **Créer**.

---

## <a name="apply-migrations"></a>Appliquer des migrations

* Exécutez l’application et sélectionnez le lien **S’inscrire**.
* Entrez l’adresse e-mail et le mot de passe du nouveau compte, puis sélectionnez **S’inscrire**.
* Suivez les instructions pour appliquer des migrations.

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

## <a name="use-secretmanager-to-store-tokens-assigned-by-login-providers"></a>Utilisez SecretManager pour stocker les jetons affectés par les fournisseurs de connexion

Les fournisseurs de connexion de réseaux sociaux affectent des jetons **ID d’application** et **Secret de l’application** lors du processus d’inscription. Les noms de jeton exacts varient selon le fournisseur. Ces jetons représentent les informations d’identification que votre application utilise pour accéder à son API. Les jetons constituent les « secrets » qui peuvent être liés à la configuration de votre application à l’aide de [Secret Manager](xref:security/app-secrets#secret-manager). Secret Manager est une alternative plus sécurisée pour stocker les jetons dans un fichier de configuration, tel que *appsettings.json*.

> [!IMPORTANT]
> Secret Manager est uniquement réservé au développement. Vous pouvez stocker et protéger les secrets de test et de production Azure avec le [fournisseur de configuration Azure Key Vault](xref:security/key-vault-configuration).

Suivez les étapes de la rubrique [Stockage sécurisé des secrets d’application lors du développement dans ASP.NET Core](xref:security/app-secrets) pour stocker les jetons affectés par chaque fournisseur de connexion ci-dessous.

## <a name="setup-login-providers-required-by-your-application"></a>Configurer les fournisseurs de connexion nécessaires à votre application

Utilisez les rubriques suivantes pour configurer votre application pour utiliser ces différents fournisseurs :

* Instructions pour [Facebook](xref:security/authentication/facebook-logins)
* Instructions pour [Twitter](xref:security/authentication/twitter-logins)
* Instructions pour [Google](xref:security/authentication/google-logins)
* Instructions pour [Microsoft](xref:security/authentication/microsoft-logins)
* Instructions pour les [autres fournisseurs](xref:security/authentication/otherlogins)

[!INCLUDE[](includes/chain-auth-providers.md)]

## <a name="optionally-set-password"></a>Définition facultative d’un mot de passe

Quand vous vous inscrivez auprès d’un fournisseur de connexion externe, vous n’avez pas de mot de passe inscrit auprès de l’application. Ceci vous évite de devoir créer et mémoriser un mot de passe pour le site, mais vous rend aussi dépendant du fournisseur de connexion externe. Si le fournisseur de connexion externe n’est pas disponible, vous ne pouvez pas vous connecter au site web.

Pour créer un mot de passe et vous connecter à l’aide de l’e-mail que vous avez défini lors du processus de connexion avec des fournisseurs externes :

* Sélectionnez le lien **Bonjour &lt;alias d’e-mail&gt;** en haut à droite pour accéder à la vue **Gérer**.

![Vue Gérer de l’application web](index/_static/pass1a.png)

* Sélectionnez **Créer**.

![Page Définir votre mot de passe](index/_static/pass2a.png)

* Définissez un mot de passe valide à utiliser pour vous connecter avec votre e-mail.

## <a name="next-steps"></a>Étapes suivantes

* Cet article a présenté l’authentification externe et expliqué les prérequis nécessaires pour ajouter des connexions externes à votre application ASP.NET Core.

* Référencez les pages spécifiques au fournisseur pour configurer les connexions pour les fournisseurs nécessaires à votre application.

* Vous souhaiterez peut-être conserver des données supplémentaires relatives à l’utilisateur et à ses jetons d’accès et d’actualisation. Pour plus d'informations, consultez <xref:security/authentication/social/additional-claims>.
