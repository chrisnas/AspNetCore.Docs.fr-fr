---
title: Tag Helper Cache dans ASP.NET Core MVC
author: pkellner
description: Découvrez comment utiliser le Tag Helper Cache.
ms.author: riande
ms.custom: mvc
ms.date: 10/10/2018
uid: mvc/views/tag-helpers/builtin-th/cache-tag-helper
ms.openlocfilehash: 0273a9805dd5db5450f57dcf3fd4d952308df074
ms.sourcegitcommit: 7a40c56bf6a6aaa63a7ee83a2cac9b3a1d77555e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/12/2019
ms.locfileid: "67856209"
---
# <a name="cache-tag-helper-in-aspnet-core-mvc"></a>Tag Helper Cache dans ASP.NET Core MVC

Par [Peter Kellner](https://peterkellner.net) et [Luke Latham](https://github.com/guardrex) 

Le Tag Helper Cache permet d’améliorer les performances de votre application ASP.NET Core en mettant en cache son contenu dans le fournisseur de caches ASP.NET Core interne.

Pour avoir une vue d’ensemble des Tag Helpers, consultez <xref:mvc/views/tag-helpers/intro>.

Le balisage Razor suivant met en cache la date actuelle :

```cshtml
<cache>@DateTime.Now</cache>
```

La première requête à la page qui contient le Tag Helper affiche la date actuelle. Les autres requêtes affichent la valeur mise en cache jusqu’à ce que le cache expire (par défaut, 20 minutes) ou jusqu’à ce que la date du cache soit supprimée du cache.

## <a name="cache-tag-helper-attributes"></a>Attributs de Tag Helper Cache

### <a name="enabled"></a>enabled

| Type d’attribut  | Exemples        | Par défaut |
| --------------- | --------------- | ------- |
| Booléen         | `true`, `false` | `true`  |

`enabled` détermine si le contenu joint par le Tag Helper Cache est mis en cache. La valeur par défaut est `true`. Si la valeur est `false`, la sortie rendue n’est **pas** mise en cache.

Exemple :

```cshtml
<cache enabled="true">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="expires-on"></a>expires-on

| Type d’attribut   | Exemples                            |
| ---------------- | ---------------------------------- |
| `DateTimeOffset` | `@new DateTime(2025,1,29,17,02,0)` |

`expires-on` définit une date d’expiration absolue pour l’élément mis en cache.

L’exemple suivant met en cache le contenu du Tag Helper Cache jusqu’à 17:02 le 29 janvier 2025 :

```cshtml
<cache expires-on="@new DateTime(2025,1,29,17,02,0)">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="expires-after"></a>expires-after

| Type d’attribut | Exemples                      | Par défaut    |
| -------------- | ---------------------------- | ---------- |
| `TimeSpan`     | `@TimeSpan.FromSeconds(120)` | 20 minutes |

`expires-after` définit la durée à partir de l’heure de la première demande pour mettre en cache le contenu.

Exemple :

```cshtml
<cache expires-after="@TimeSpan.FromSeconds(120)">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

Le moteur de vue Razor définit la valeur par défaut `expires-after` sur vingt minutes.

### <a name="expires-sliding"></a>expires-sliding

| Type d’attribut | Exemples                     |
| -------------- | --------------------------- |
| `TimeSpan`     | `@TimeSpan.FromSeconds(60)` |

Définit l’heure à laquelle une entrée de cache doit être supprimée si sa valeur n’a fait l’objet d’aucun accès.

Exemple :

```cshtml
<cache expires-sliding="@TimeSpan.FromSeconds(60)">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-header"></a>vary-by-header

| Type d’attribut | Exemples                                    |
| -------------- | ------------------------------------------- |
| Chaîne         | `User-Agent`, `User-Agent,content-encoding` |

`vary-by-header` accepte une liste séparée par des virgules de valeurs d’en-tête qui déclenchent une actualisation du cache quand elles changent.

L’exemple suivant analyse la valeur d’en-tête `User-Agent`. L’exemple met en cache le contenu de chaque valeur `User-Agent` différente présentée au serveur web :

```cshtml
<cache vary-by-header="User-Agent">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-query"></a>vary-by-query

| Type d’attribut | Exemples             |
| -------------- | -------------------- |
| Chaîne         | `Make`, `Make,Model` |

`vary-by-query` accepte une liste de <xref:Microsoft.AspNetCore.Http.IQueryCollection.Keys*> séparées par des virgules dans une chaîne de requête (<xref:Microsoft.AspNetCore.Http.HttpRequest.Query*>) déclenchant une actualisation du cache quand la valeur de l’une des clés répertoriées est modifiée.

L’exemple suivant analyse les valeurs de `Make` et `Model`. L’exemple met en cache le contenu de chaque valeur `Make` et `Model` différente présentée au serveur web :

```cshtml
<cache vary-by-query="Make,Model">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-route"></a>vary-by-route

| Type d’attribut | Exemples             |
| -------------- | -------------------- |
| Chaîne         | `Make`, `Make,Model` |

`vary-by-route` accepte une liste séparée par des virgules de noms de paramètre de route qui déclenchent une actualisation du cache quand la valeur du paramètre des données de route change.

Exemple :

*Startup.cs* :

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{Make?}/{Model?}");
```

*Index.cshtml*:

```cshtml
<cache vary-by-route="Make,Model">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-cookie"></a>vary-by-cookie

| Type d’attribut | Exemples                                                                         |
| -------------- | -------------------------------------------------------------------------------- |
| Chaîne         | `.AspNetCore.Identity.Application`, `.AspNetCore.Identity.Application,HairColor` |

`vary-by-cookie` accepte une liste séparée par des virgules de noms de cookie qui déclenchent une actualisation du cache quand la valeur du cookie change.

L’exemple suivant analyse le cookie associé à ASP.NET Core Identity. Lorsqu’un utilisateur est authentifié, une modification dans le cookie Identity déclenche une actualisation du cache :

```cshtml
<cache vary-by-cookie=".AspNetCore.Identity.Application">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-user"></a>vary-by-user

| Type d’attribut  | Exemples        | Par défaut |
| --------------- | --------------- | ------- |
| Booléen         | `true`, `false` | `true`  |

`vary-by-user` spécifie si le cache se réinitialise ou pas quand l’utilisateur connecté (ou principal du contexte) change. L’utilisateur actuel est également connu comme principal du contexte de la demande et peut être affiché dans une vue Razor en référençant `@User.Identity.Name`.

L’exemple suivant analyse l’utilisateur actuellement connecté pour déclencher une actualisation du cache :

```cshtml
<cache vary-by-user="true">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

L’utilisation de cet attribut permet de conserver le contenu dans le cache lors d’un cycle de connexion et de déconnexion. Lorsque la valeur est définie sur `true`, un cycle d’authentification invalide le cache pour l’utilisateur authentifié. Le cache est invalidé, car une nouvelle valeur de cookie unique est générée quand un utilisateur est authentifié. Le cache est conservé pour l’état anonyme quand aucun cookie n’est présent ou quand le cookie a expiré. Si l’utilisateur n’est **pas** authentifié, le cache est conservé.

### <a name="vary-by"></a>vary-by

| Type d’attribut | Exemples  |
| -------------- | -------- |
| Chaîne         | `@Model` |

`vary-by` permet la personnalisation des données mises en cache. Quand l’objet référencé par la valeur de la chaîne de l’attribut change, le contenu du Tag Helper Cache est mis à jour. Souvent, une concaténation de chaîne des valeurs de modèle est affectée à cet attribut. En effet, cela entraîne un scénario où une mise à jour de l’une des valeurs concaténées invalide le cache.

L’exemple suivant suppose que la méthode de contrôleur restituant la vue additionne la valeur entière des deux paramètres de routage, `myParam1` et `myParam2`, et retourne la somme en tant que propriété de modèle unique. Quand cette somme est modifiée, le contenu du Tag Helper Cache est rendu et mis en cache à nouveau.  

Action :

```csharp
public IActionResult Index(string myParam1, string myParam2, string myParam3)
{
    int num1;
    int num2;
    int.TryParse(myParam1, out num1);
    int.TryParse(myParam2, out num2);
    return View(viewName, num1 + num2);
}
```

*Index.cshtml*:

```cshtml
<cache vary-by="@Model">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="priority"></a>priority

| Type d’attribut      | Exemples                               | Par défaut  |
| ------------------- | -------------------------------------- | -------- |
| `CacheItemPriority` | `High`, `Low`, `NeverRemove`, `Normal` | `Normal` |

`priority` fournit des instructions de suppression de cache au fournisseur de caches intégré. Le serveur web supprime d’abord les entrées de cache `Low` en cas de sollicitation de la mémoire.

Exemple :

```cshtml
<cache priority="High">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

L’attribut `priority` ne garantit pas un niveau spécifique de rétention du cache. `CacheItemPriority` est uniquement une suggestion. Si vous affectez à cet attribut la valeur `NeverRemove`, il n’est pas garanti que les éléments du cache soient toujours conservés. Pour plus d’informations, consultez les rubriques de la section [Ressources supplémentaires](#additional-resources).

Le Tag Helper Cache dépend du [service de cache en mémoire](xref:performance/caching/memory). Le Tag Helper Cache ajoute le service s’il ne l’a pas encore été.

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:performance/caching/memory>
* <xref:security/authentication/identity>
