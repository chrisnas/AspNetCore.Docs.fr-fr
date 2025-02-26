---
title: Confirmation de compte et récupération de mot de passe dans ASP.NET Core
author: rick-anderson
description: Découvrez comment créer une application ASP.NET Core avec une confirmation par e-mail et une réinitialisation du mot de passe.
ms.author: riande
ms.date: 03/11/2019
uid: security/authentication/accconfirm
ms.openlocfilehash: 8a515990be584aa1233fc3bf77811ae3784d9b1c
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/18/2019
ms.locfileid: "71081560"
---
# <a name="account-confirmation-and-password-recovery-in-aspnet-core"></a>Confirmation de compte et récupération de mot de passe dans ASP.NET Core

Par [Rick Anderson](https://twitter.com/RickAndMSFT), [Ponant](https://github.com/Ponant)et [Joe Audette](https://twitter.com/joeaudette)

Ce didacticiel montre comment créer une application ASP.NET Core avec une confirmation par e-mail et une réinitialisation du mot de passe. Ce didacticiel n'est **pas** un sujet pour débuter. Vous devez être familiarisé avec :

* [ASP.NET Core](xref:tutorials/razor-pages/razor-pages-start)
* [Authentification](xref:security/authentication/identity)
* [Entity Framework Core](xref:data/ef-mvc/intro)

<!-- see C:/Dropbox/wrk/Code/SendGridConsole/Program.cs -->

::: moniker range="<= aspnetcore-2.0"

Consultez [ce fichier PDF](https://webpifeed.blob.core.windows.net/webpifeed/Partners/asp.net_repo_pdf_1-16-18.pdf) pour obtenir la version ASP.net Core 1,1.

::: moniker-end

::: moniker range="> aspnetcore-2.2"

## <a name="prerequisites"></a>Prérequis

[.NET Core 3,0 SDK ou version ultérieure](https://dotnet.microsoft.com/download/dotnet-core/3.0)

## <a name="create-and-test-a-web-app-with-authentication"></a>Créer et tester une application Web avec l’authentification

Exécutez les commandes suivantes pour créer une application Web avec l’authentification.

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet run
```

Exécutez l’application, sélectionnez le lien **S'inscrire** et inscrivez un utilisateur. Une fois inscrit, vous êtes redirigé vers la page `/Identity/Account/RegisterConfirmation` vers qui contient un lien pour simuler la confirmation de l’e-mail :

* Sélectionnez le `Click here to confirm your account` lien.
* Sélectionnez le lien de **connexion** et connectez-vous avec les mêmes informations d’identification.
* Sélectionnez le `Hello YourEmail@provider.com!` lien, qui vous redirige vers la `/Identity/Account/Manage/PersonalData` page.
* Sélectionnez l’onglet **données personnelles** sur la gauche, puis sélectionnez **supprimer**.

### <a name="configure-an-email-provider"></a>Configurer un fournisseur de messagerie

Dans ce didacticiel, [SendGrid](https://sendgrid.com) est utilisé pour envoyer des messages électroniques. Vous avez besoin d’un compte SendGrid et une clé pour envoyer un courrier électronique. Vous pouvez utiliser d’autres fournisseurs de messagerie. Nous vous recommandons d'utiliser SendGrid ou un autre service de messagerie pour envoyer un courrier électronique. SMTP est difficile à sécuriser et à correctement configurer.

Créez une classe pour extraire la clé de messagerie électronique sécurisée. Pour cet exemple, créez *services/AuthMessageSenderOptions. cs*:

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a>Configurer des secrets d’utilisateur SendGrid

Définissez `SendGridUser` et `SendGridKey` avec l'[outil Gestionnaire de secret](xref:security/app-secrets). Exemple :

```dotnetcli
dotnet user-secrets set SendGridUser RickAndMSFT
dotnet user-secrets set SendGridKey <key>

Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

Sur Windows, le gestionnaire de secret stocke des paires de clés/valeur dans un fichier *secrets.json* dans le répertoire `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>`.

Le contenu du fichier *secrets.json* n’est pas chiffré. Le balisage suivant montre le fichier *secrets. JSON* . La `SendGridKey` valeur a été supprimée.

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

Pour plus d’informations, consultez le [modèle d’options](xref:fundamentals/configuration/options) et la [configuration](xref:fundamentals/configuration/index).

### <a name="install-sendgrid"></a>Installer SendGrid

Ce didacticiel montre comment ajouter des notifications par courrier électronique via [SendGrid](https://sendgrid.com/), mais vous pouvez envoyer des e-mails en utilisant SMTP et d'autres mécanismes.

Installer le package NuGet `SendGrid` :

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

À partir de la console du gestionnaire de package, entrez la commande suivante :

```powershell
Install-Package SendGrid
```

# <a name="net-core-clitabnetcore-cli"></a>[CLI .NET Core](#tab/netcore-cli)

À partir de la console, entrez la commande suivante :

```dotnetcli
dotnet add package SendGrid
```

---

Consultez [Débuter avec SendGrid gratuitement](https://sendgrid.com/free/) pour vous inscrire pour un compte SendGrid gratuit.

### <a name="implement-iemailsender"></a>Implémenter IEmailSender

Pour implémenter `IEmailSender`, créez *services/EMailSender. cs* avec du code similaire à ce qui suit :

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a>Configurer le démarrage pour la prise en charge de la messagerie

Ajoutez le code suivant à la `ConfigureServices` méthode dans le fichier *Startup.cs* :

* Ajoutez `EmailSender` en tant que service temporaire.
* Inscrivez l' `AuthMessageSenderOptions` instance de configuration.

[!code-csharp[](accconfirm/sample/WebPWrecover30/Startup.cs?name=snippet1&highlight=11-15)]

## <a name="register-confirm-email-and-reset-password"></a>Inscrire, confirmer l’adresse e-mail et réinitialiser le mot de passe

Exécuter l’application web, et testez le flux de confirmation du compte et de récupération du mot de passe.

* Exécutez l’application et inscrire un nouvel utilisateur
* Recherchez le lien de confirmation du compte dans votre messagerie. Consultez [Déboguer la messagerie](#debug) si vous ne recevez pas l’e-mail.
* Cliquez sur le lien pour confirmer votre adresse de messagerie.
* Connectez-vous avec votre adresse de messagerie et votre mot de passe.
* Déconnectez-vous.

### <a name="test-password-reset"></a>Tester la réinitialisation du mot de passe

* Si vous êtes connecté, sélectionnez **déconnexion**.
* Sélectionnez le lien **Connexion** et sélectionnez le lien **Vous avez oublié votre mot de passe ?** .
* Entrez l’adresse de messagerie que vous avez utilisée pour inscrire le compte.
* Un e-mail contenant un lien pour réinitialiser votre mot de passe est envoyé. Vérifiez votre adresse de messagerie, puis cliquez sur le lien pour réinitialiser votre mot de passe. Une fois que votre mot de passe a été réinitialisé avec succès, vous pouvez vous connecter avec votre adresse de messagerie et votre nouveau mot de passe.

## <a name="change-email-and-activity-timeout"></a>Modifier le délai d’expiration de l’e-mail et de l’activité

Le délai d’inactivité par défaut est de 14 jours. Le code suivant définit le délai d’expiration d’inactivité sur 5 jours :

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a>Modifier toutes les durées de vie des jetons de protection des données

Le code suivant modifie le délai d’expiration de tous les jetons de protection des données à 3 heures :

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAllTokens.cs?name=snippet1&highlight=11-12)]

Les jetons utilisateur d’identité intégrés (consultez [AspNetCore/SRC/Identity/extensions. Core/SRC/TokenOptions. cs](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) ) ont un [délai d’expiration d’un jour](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).

### <a name="change-the-email-token-lifespan"></a>Modifier la durée de vie du jeton d’e-mail

La durée de vie des jetons par défaut des [jetons d’utilisateur d’identité](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) est d' [un jour](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs). Cette section montre comment modifier la durée de vie des jetons de courrier électronique.

Ajoutez un [> TUser\<DataProtectorTokenProvider](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1) personnalisé et <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions>:

[!code-csharp[](accconfirm/sample/WebPWrecover30/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

Ajoutez le fournisseur personnalisé au conteneur de service :

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupEmail.cs?name=snippet1&highlight=10-16)]

### <a name="resend-email-confirmation"></a>Renvoyer l’e-mail de confirmation

Consultez [ce problème GitHub](https://github.com/aspnet/AspNetCore/issues/5410).

<a name="debug"></a>

### <a name="debug-email"></a>Message de débogage

Si vous ne parvenez à faire fonctionner l'email :

* Définissez un point d' `EmailSender.Execute` arrêt dans `SendGridClient.SendEmailAsync` pour vérifier que est appelé.
* Créez une [application console pour envoyer du courrier électronique](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) à l' `EmailSender.Execute`aide d’un code similaire à.
* Examinez la page [Activité de la messagerie](https://sendgrid.com/docs/User_Guide/email_activity.html) page.
* Vérifiez votre dossier de courrier indésirable.
* Essayez un autre alias de messagerie sur un autre fournisseur de messagerie (Microsoft, Yahoo, Gmail, etc.)
* Essayez d’envoyer à différents comptes de messagerie.

Une **bonne pratique de sécurité** consiste à **ne pas** utiliser des secrets de production en développement et en test. Si vous publiez l’application sur Azure, définissez les secrets SendGrid en tant que paramètres d’application dans le portail d’application Web Azure. Le système de configuration est conçu pour lire les clés à partir de variables d’environnement.

## <a name="combine-social-and-local-login-accounts"></a>Combiner les comptes de connexion sociale et locale

Pour compléter cette section, vous devez d’abord activer un fournisseur d’authentification externe. Consultez [l’authentification Facebook, Google et fournisseur externe](xref:security/authentication/social/index).

Vous pouvez combiner des comptes locaux et de réseaux sociaux en cliquant sur le lien de votre e-mail. Dans la séquence suivante, "RickAndMSFT@gmail.com" est d’abord créé en tant que connexion locale ; cependant, vous pouvez d’abord créer le compte en tant que connexion de réseau social, puis ajouter une connexion locale.

![Application Web : RickAndMSFT@gmail.com utilisateur authentifié](accconfirm/_static/rick.png)

Cliquez sur le lien **Gérer**. Notez la valeur 0 externe (connexions sociales) associée à ce compte.

![Gérer l’affichage](accconfirm/_static/manage.png)

Cliquez sur le lien vers un autre service de connexion et acceptez les demandes d’application. Dans l’image suivante, Facebook est le fournisseur d’authentification externe :

![Gérer l’affichage de vos connexions externes à l’objet Facebook](accconfirm/_static/fb.png)

Les deux comptes ont été combinés. Vous pouvez vous connecter avec l’un ou l’autre de ces comptes. Vous pouvez faire en sorte que vos utilisateurs ajoutent des comptes locaux au cas où leur service d’authentification de connexion sociale soit indisponible ou que, plus probablement, ils aient perdu l’accès à leur compte social.

## <a name="enable-account-confirmation-after-a-site-has-users"></a>Activer la confirmation du compte une fois qu’un site a des utilisateurs

Activer la confirmation du compte sur un site avec des utilisateurs bloque tous les utilisateurs existants. Ceux-ci sont bloqués, car leurs comptes ne sont pas confirmés. Pour contourner le blocage des utilisateurs existants, utilisez une des approches suivantes :

* Mise à jour de la base de données pour marquer tous les utilisateurs existants comme étant confirmés.
* Confirmez les utilisateurs existants. Par exemple, en envoyant par lot des e-mails avec des liens de confirmation.

::: moniker-end

::: moniker range="> aspnetcore-2.0 < aspnetcore-3.0"

## <a name="prerequisites"></a>Prérequis

[.NET Core 2,2 SDK ou version ultérieure](https://www.microsoft.com/net/download/all)

## <a name="create-a-web--app-and-scaffold-identity"></a>Créer une application Web et une identité de structure

Exécutez les commandes suivantes pour créer une application Web avec l’authentification.

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator identity -dc WebPWrecover.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout;Account.ConfirmEmail"
dotnet ef database drop -f
dotnet ef database update
dotnet run

```

## <a name="test-new-user-registration"></a>Tester l’inscription d’un nouvel utilisateur

Exécutez l’application, sélectionnez le lien **S'inscrire** et inscrivez un utilisateur. À ce stade, la seule validation sur l’adresse e-mail se fait avec l'attribut [[EmailAddress]](/dotnet/api/system.componentmodel.dataannotations.emailaddressattribute). Après avoir soumis l’inscription, vous êtes connecté à l’application. Plus loin dans ce didacticiel, le code est mis à jour afin que les nouveaux utilisateurs ne puissent pas se connecter tant que leur adresse de messagerie n’a pas été validée.

[!INCLUDE[](~/includes/view-identity-db.md)]

Notez que le champ `EmailConfirmed` de la table est à `False`.

Vous pouvez souhaiter réutiliser cet e-mail à l’étape suivante quand l’application envoie un e-mail de confirmation. Cliquez avec le bouton droit sur la ligne, puis sélectionnez **Supprimer**. La suppression de l’alias d’e-mail facilite les étapes suivantes.

<a name="prevent-login-at-registration"></a>

## <a name="require-email-confirmation"></a>Demander une confirmation par courrier électronique

Il est recommandé de confirmer l’adresse de messagerie d’un nouvel enregistrement d’utilisateur. Envoyer un courrier électronique de confirmation permet de vérifier qu’ils n'empruntent pas l'identité d'une autre personne (autrement dit, ils ne se sont pas inscrits avec l'adresse de messageriedquelqu'un d’autre). Supposons que vous ayez un forum de discussion, et que vous souhaitiez empêcher "yli@example.com" de s’inscription en tant que "nolivetto@contoso.com". Sans la confirmation par courrier électronique, "nolivetto@contoso.com" peut recevoir un e-mail indésirable de votre application. Supposons que l’utilisateur s'inscrit par inadvertance en tant que "ylo@example.com" et que vous n’avez pas remarqué la faute d’orthographe de "yli". Ils ne pourraient pas utiliser la récupération de mot de passe, car l’application n’a pas leur courrier électronique correct. La confirmation par e-mail fournit une protection limitée des robots. L'email de confirmation ne fournit pas une protection contre les utilisateurs malveillants avec plusieurs comptes de messagerie.

En règle générale, vous souhaitez empêcher les nouveaux utilisateurs d'envoyer des données à votre site web tant que leur adresse e-mail n’est pas confirmée.

Mise `Startup.ConfigureServices` à jour pour exiger un e-mail confirmé :

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=8-11)]

`config.SignIn.RequireConfirmedEmail = true;` empêche les utilisateurs enregistrés de se connecter jusqu'à ce que leur adresse de messagerie soit confirmée.

### <a name="configure-email-provider"></a>Configurer le fournisseur de messagerie

Dans ce didacticiel, [SendGrid](https://sendgrid.com) est utilisé pour envoyer des messages électroniques. Vous avez besoin d’un compte SendGrid et une clé pour envoyer un courrier électronique. Vous pouvez utiliser d’autres fournisseurs de messagerie. ASP.NET Core 2.x inclut `System.Net.Mail`, qui vous permet d’envoyer par courrier électronique à partir de votre application. Nous vous recommandons d'utiliser SendGrid ou un autre service de messagerie pour envoyer un courrier électronique. SMTP est difficile à sécuriser et à correctement configurer.

Créez une classe pour extraire la clé de messagerie électronique sécurisée. Pour cet exemple, créez *services/AuthMessageSenderOptions. cs*:

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a>Configurer des secrets d’utilisateur SendGrid

Définissez `SendGridUser` et `SendGridKey` avec l'[outil Gestionnaire de secret](xref:security/app-secrets). Exemple :

```console
C:/WebAppl>dotnet user-secrets set SendGridUser RickAndMSFT
info: Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

Sur Windows, le gestionnaire de secret stocke des paires de clés/valeur dans un fichier *secrets.json* dans le répertoire `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>`.

Le contenu du fichier *secrets.json* n’est pas chiffré. Le balisage suivant montre le fichier *secrets. JSON* . La `SendGridKey` valeur a été supprimée.

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

Pour plus d’informations, consultez le [modèle d’options](xref:fundamentals/configuration/options) et la [configuration](xref:fundamentals/configuration/index).

### <a name="install-sendgrid"></a>Installer SendGrid

Ce didacticiel montre comment ajouter des notifications par courrier électronique via [SendGrid](https://sendgrid.com/), mais vous pouvez envoyer des e-mails en utilisant SMTP et d'autres mécanismes.

Installer le package NuGet `SendGrid` :

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

À partir de la console du gestionnaire de package, entrez la commande suivante :

```powershell
Install-Package SendGrid
```

# <a name="net-core-clitabnetcore-cli"></a>[CLI .NET Core](#tab/netcore-cli)

À partir de la console, entrez la commande suivante :

```dotnetcli
dotnet add package SendGrid
```

---

Consultez [Débuter avec SendGrid gratuitement](https://sendgrid.com/free/) pour vous inscrire pour un compte SendGrid gratuit.

### <a name="implement-iemailsender"></a>Implémenter IEmailSender

Pour implémenter `IEmailSender`, créez *services/EMailSender. cs* avec du code similaire à ce qui suit :

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a>Configurer le démarrage pour la prise en charge de la messagerie

Ajoutez le code suivant à la `ConfigureServices` méthode dans le fichier *Startup.cs* :

* Ajoutez `EmailSender` en tant que service temporaire.
* Inscrivez l' `AuthMessageSenderOptions` instance de configuration.

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=15-99)]

## <a name="enable-account-confirmation-and-password-recovery"></a>Activer la récupération du mot de passe et la confirmation du compte

Le modèle contient du code pour la confirmation du compte et la récupération de mot de passe. Recherchez la `OnPostAsync` méthode dans *Areas/Identity/pages/Account/Register. cshtml. cs*.

Empêchez les utilisateurs nouvellement inscrits d’être automatiquement connectés en commentant la ligne suivante :

```csharp
await _signInManager.SignInAsync(user, isPersistent: false);
```

La méthode complète est montrée avec la ligne modifiée mise en surbrillance :

[!code-csharp[](accconfirm/sample/WebPWrecover22/Areas/Identity/Pages/Account/Register.cshtml.cs?highlight=22&name=snippet_Register)]

## <a name="register-confirm-email-and-reset-password"></a>Inscrire, confirmer l’adresse e-mail et réinitialiser le mot de passe

Exécuter l’application web, et testez le flux de confirmation du compte et de récupération du mot de passe.

* Exécutez l’application et inscrire un nouvel utilisateur
* Recherchez le lien de confirmation du compte dans votre messagerie. Consultez [Déboguer la messagerie](#debug) si vous ne recevez pas l’e-mail.
* Cliquez sur le lien pour confirmer votre adresse de messagerie.
* Connectez-vous avec votre adresse de messagerie et votre mot de passe.
* Déconnectez-vous.

### <a name="view-the-manage-page"></a>Afficher la page gérer

Sélectionnez votre nom d’utilisateur dans la fenêtre ![navigateur : navigateur avec le nom d’utilisateur](accconfirm/_static/un.png)

La page de gestion s’affiche avec l'onglet **profil** sélectionné. L'**Email** affiche une case à cocher indiquant que l’adresse de messagerie a été confirmée.

### <a name="test-password-reset"></a>Tester la réinitialisation du mot de passe

* Si vous êtes connecté, sélectionnez **déconnexion**.
* Sélectionnez le lien **Connexion** et sélectionnez le lien **Vous avez oublié votre mot de passe ?** .
* Entrez l’adresse de messagerie que vous avez utilisée pour inscrire le compte.
* Un e-mail contenant un lien pour réinitialiser votre mot de passe est envoyé. Vérifiez votre adresse de messagerie, puis cliquez sur le lien pour réinitialiser votre mot de passe. Une fois que votre mot de passe a été réinitialisé avec succès, vous pouvez vous connecter avec votre adresse de messagerie et votre nouveau mot de passe.

## <a name="change-email-and-activity-timeout"></a>Modifier le délai d’expiration de l’e-mail et de l’activité

Le délai d’inactivité par défaut est de 14 jours. Le code suivant définit le délai d’expiration d’inactivité sur 5 jours :

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a>Modifier toutes les durées de vie des jetons de protection des données

Le code suivant modifie le délai d’expiration de tous les jetons de protection des données à 3 heures :

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAllTokens.cs?name=snippet1&highlight=15-16)]

Les jetons utilisateur d’identité intégrés (consultez [AspNetCore/SRC/Identity/extensions. Core/SRC/TokenOptions. cs](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) ) ont un [délai d’expiration d’un jour](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).

### <a name="change-the-email-token-lifespan"></a>Modifier la durée de vie du jeton d’e-mail

La durée de vie des jetons par défaut des [jetons d’utilisateur d’identité](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) est d' [un jour](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs). Cette section montre comment modifier la durée de vie des jetons de courrier électronique.

Ajoutez un [> TUser\<DataProtectorTokenProvider](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1) personnalisé et <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions>:

[!code-csharp[](accconfirm/sample/WebPWrecover22/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

Ajoutez le fournisseur personnalisé au conteneur de service :

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupEmail.cs?name=snippet1&highlight=10-13,18)]

### <a name="resend-email-confirmation"></a>Renvoyer l’e-mail de confirmation

Consultez [ce problème GitHub](https://github.com/aspnet/AspNetCore/issues/5410).

<a name="debug"></a>

### <a name="debug-email"></a>Message de débogage

Si vous ne parvenez à faire fonctionner l'email :

* Définissez un point d' `EmailSender.Execute` arrêt dans `SendGridClient.SendEmailAsync` pour vérifier que est appelé.
* Créez une [application console pour envoyer du courrier électronique](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) à l' `EmailSender.Execute`aide d’un code similaire à.
* Examinez la page [Activité de la messagerie](https://sendgrid.com/docs/User_Guide/email_activity.html) page.
* Vérifiez votre dossier de courrier indésirable.
* Essayez un autre alias de messagerie sur un autre fournisseur de messagerie (Microsoft, Yahoo, Gmail, etc.)
* Essayez d’envoyer à différents comptes de messagerie.

Une **bonne pratique de sécurité** consiste à **ne pas** utiliser des secrets de production en développement et en test. Si vous publiez l’application sur Azure, vous pouvez définir les secrets SendGrid en tant que paramètres d’application dans le portail Azure Web App. Le système de configuration est conçu pour lire les clés à partir de variables d’environnement.

## <a name="combine-social-and-local-login-accounts"></a>Combiner les comptes de connexion sociale et locale

Pour compléter cette section, vous devez d’abord activer un fournisseur d’authentification externe. Consultez [l’authentification Facebook, Google et fournisseur externe](xref:security/authentication/social/index).

Vous pouvez combiner des comptes locaux et de réseaux sociaux en cliquant sur le lien de votre e-mail. Dans la séquence suivante, "RickAndMSFT@gmail.com" est d’abord créé en tant que connexion locale ; cependant, vous pouvez d’abord créer le compte en tant que connexion de réseau social, puis ajouter une connexion locale.

![Application Web : RickAndMSFT@gmail.com utilisateur authentifié](accconfirm/_static/rick.png)

Cliquez sur le lien **Gérer**. Notez la valeur 0 externe (connexions sociales) associée à ce compte.

![Gérer l’affichage](accconfirm/_static/manage.png)

Cliquez sur le lien vers un autre service de connexion et acceptez les demandes d’application. Dans l’image suivante, Facebook est le fournisseur d’authentification externe :

![Gérer l’affichage de vos connexions externes à l’objet Facebook](accconfirm/_static/fb.png)

Les deux comptes ont été combinés. Vous pouvez vous connecter avec l’un ou l’autre de ces comptes. Vous pouvez faire en sorte que vos utilisateurs ajoutent des comptes locaux au cas où leur service d’authentification de connexion sociale soit indisponible ou que, plus probablement, ils aient perdu l’accès à leur compte social.

## <a name="enable-account-confirmation-after-a-site-has-users"></a>Activer la confirmation du compte une fois qu’un site a des utilisateurs

Activer la confirmation du compte sur un site avec des utilisateurs bloque tous les utilisateurs existants. Ceux-ci sont bloqués, car leurs comptes ne sont pas confirmés. Pour contourner le blocage des utilisateurs existants, utilisez une des approches suivantes :

* Mise à jour de la base de données pour marquer tous les utilisateurs existants comme étant confirmés.
* Confirmez les utilisateurs existants. Par exemple, en envoyant par lot des e-mails avec des liens de confirmation.

::: moniker-end
