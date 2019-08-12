---
title: Créer et utiliser des composants ASP.NET Core Razor
author: guardrex
description: Découvrez comment créer et utiliser des composants Razor, notamment comment lier des données, gérer des événements et gérer des cycles de vie de composant.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 08/02/2019
uid: blazor/components
ms.openlocfilehash: 43457bffd748ebba68cc86d33fdeb98dc419704b
ms.sourcegitcommit: 776367717e990bdd600cb3c9148ffb905d56862d
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/09/2019
ms.locfileid: "68948429"
---
# <a name="create-and-use-aspnet-core-razor-components"></a>Créer et utiliser des composants ASP.NET Core Razor

Par [Luke Latham](https://github.com/guardrex) et [Daniel Roth](https://github.com/danroth27)

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

Les applications éblouissantes sont créées à l’aide de *composants*. Un composant est un bloc d’interface utilisateur (IU) autonome, tel qu’une page, une boîte de dialogue ou un formulaire. Un composant comprend le balisage HTML et la logique de traitement requise pour injecter des données ou répondre à des événements d’interface utilisateur. Les composants sont flexibles et légers. Elles peuvent être imbriquées, réutilisées et partagées entre les projets.

## <a name="component-classes"></a>Classes de composant

Les composants sont implémentés dans les fichiers de composants [Razor](xref:mvc/views/razor) ( *. Razor*) C# à l’aide d’une combinaison de balises et html. Un composant de éblouissant est officiellement désigné sous le terme de *composant Razor*.

Les composants peuvent être créés à l’aide de l’extension de fichier *. cshtml* . Utilisez la `_RazorComponentInclude` propriété MSBuild dans le fichier projet pour identifier les fichiers *. cshtml* du composant. Par exemple, une application qui spécifie que tous les fichiers *. cshtml* sous le dossier *pages* doivent être traités comme des fichiers de composants Razor:

```xml
<PropertyGroup>
  <_RazorComponentInclude>Pages\**\*.cshtml</_RazorComponentInclude>
</PropertyGroup>
```

L’interface utilisateur d’un composant est définie à l’aide de HTML. La logique de rendu dynamique (par exemple, les boucles, conditions, expressions) est ajoutée à l’aide d’une syntaxe C# intégrée appelée [Razor](xref:mvc/views/razor). Quand une application est compilée, le balisage C# html et la logique de rendu sont convertis en une classe de composant. Le nom de la classe générée correspond au nom du fichier.

Les membres de la classe de composants sont définis dans un bloc `@code`. Dans le `@code` bloc, l’état du composant (propriétés, champs) est spécifié avec des méthodes pour la gestion des événements ou pour la définition d’une autre logique de composant. Plus d’un `@code` bloc est autorisé.

> [!NOTE]
> Dans les préversions précédentes de ASP.net Core 3,0 `@functions` , les blocs étaient utilisés dans les mêmes `@code` buts que les blocs dans les composants Razor. `@functions`les blocs continuent de fonctionner dans les composants Razor, mais nous vous `@code` recommandons d’utiliser le bloc dans ASP.net Core 3,0 Preview 6 ou version ultérieure.

Les membres de composant peuvent être utilisés dans le cadre de la logique de C# rendu du composant à `@`l’aide d’expressions qui commencent par. Par exemple, un C# champ est rendu en préfixant `@` le nom du champ. L’exemple suivant évalue et affiche:

* `_headingFontStyle`à la valeur de propriété CSS `font-style`pour.
* `_headingText`au contenu de l' `<h1>` élément.

```cshtml
<h1 style="font-style:@_headingFontStyle">@_headingText</h1>

@code {
    private string _headingFontStyle = "italic";
    private string _headingText = "Put on your new Blazor!";
}
```

Une fois le composant restitué initialement, le composant régénère son arborescence de rendu en réponse aux événements. Il compare ensuite la nouvelle arborescence de rendu à la précédente et applique toutes les modifications à la Document Object Model du navigateur (DOM).

Les composants sont C# des classes ordinaires qui peuvent être placées n’importe où dans un projet. Les composants qui produisent des pages Web résident généralement dans le dossier *pages* . Les composants qui ne sont pas des pages sont souvent placés dans le dossier *partagé* ou dans un dossier personnalisé ajouté au projet. Pour utiliser un dossier personnalisé, ajoutez l’espace de noms du dossier personnalisé au composant parent ou au fichier *_Imports. Razor* de l’application. Par exemple, l’espace de noms suivant rend les composants dans un dossier de *composants* disponibles lorsque l’espace `WebApplication`de noms racine de l’application est:

```cshtml
@using WebApplication.Components
```

## <a name="integrate-components-into-razor-pages-and-mvc-apps"></a>Intégrer des composants dans des applications Razor Pages et MVC

Utilisez des composants avec des applications Razor Pages et MVC existantes. Il n’est pas nécessaire de réécrire des pages ou des vues existantes pour utiliser des composants Razor. Lorsque la page ou la vue est restituée, les composants sont prérendus en même temps.

Pour afficher un composant à partir d’une page ou d’une `RenderComponentAsync<TComponent>` vue, utilisez la méthode d’assistance HTML:

```cshtml
<div id="Counter">
    @(await Html.RenderComponentAsync<Counter>(new { IncrementAmount = 10 }))
</div>
```

Alors que les pages et les vues peuvent utiliser des composants, la réciproque n’est pas vraie. Les composants ne peuvent pas utiliser les scénarios spécifiques aux vues et aux pages, tels que les vues partielles et les sections. Pour utiliser la logique de la vue partielle dans un composant, factorisez la logique de la vue partielle dans un composant.

Pour plus d’informations sur la façon dont les composants sont rendus et l’état des composants géré dans les applications côté <xref:blazor/hosting-models> serveur éblouissantes, consultez l’article.

## <a name="using-components"></a>Utilisation des composants

Les composants peuvent inclure d’autres composants en les déclarant à l’aide de la syntaxe d’élément HTML. Le balisage pour l’utilisation d’un composant ressemble à une balise HTML où le nom de la balise est le type du composant.

Le balisage suivant dans *index. Razor* rend une `HeadingComponent` instance de:

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/Index.razor?name=snippet_HeadingComponent)]

*Composants/HeadingComponent. Razor*:

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/HeadingComponent.razor)]

## <a name="component-parameters"></a>Paramètres de composant

Les composants peuvent avoir des *paramètres de composant*, qui sont définis à l’aide de propriétés (généralement *non publiques*) sur `[Parameter]` la classe de composant avec l’attribut. Utilisez des attributs pour spécifier des arguments pour un composant dans le balisage.

*Composants/ChildComponent. Razor*:

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/ChildComponent.razor?highlight=11-12)]

Dans l’exemple suivant, le `ParentComponent` définit la valeur de la `Title` propriété de `ChildComponent`.

*Pages/ParentComponent. Razor*:

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=5-6)]

## <a name="child-content"></a>Contenu enfant

Les composants peuvent définir le contenu d’un autre composant. Le composant d’affectation fournit le contenu entre les balises qui spécifient le composant récepteur.

Dans l’exemple suivant, `ChildComponent` a une `ChildContent` propriété qui représente un `RenderFragment`, qui représente un segment de l’interface utilisateur à restituer. La valeur de `ChildContent` est positionnée dans le balisage du composant où le contenu doit être rendu. La valeur de `ChildContent` est reçue du composant parent et rendue à l’intérieur du panneau de `panel-body`démarrage.

*Composants/ChildComponent. Razor*:

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/ChildComponent.razor?highlight=3,14-15)]

> [!NOTE]
> La propriété qui reçoit `RenderFragment` le contenu doit être `ChildContent` nommée par Convention.

Les éléments `ParentComponent` suivants peuvent fournir du contenu pour `ChildComponent` le rendu de en plaçant le `<ChildComponent>` contenu à l’intérieur des balises.

*Pages/ParentComponent. Razor*:

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=7-8)]

## <a name="attribute-splatting-and-arbitrary-parameters"></a>Réprojection d’attribut et paramètres arbitraires

Les composants peuvent capturer et restituer des attributs supplémentaires en plus des paramètres déclarés du composant. Des attributs supplémentaires peuvent être capturés dans un dictionnaire , puis réintégrés à un élément lorsque le composant est rendu [@attributes](xref:mvc/views/razor#attributes) à l’aide de la directive Razor. Ce scénario est utile lors de la définition d’un composant qui produit un élément de balisage qui prend en charge diverses personnalisations. Par exemple, il peut être fastidieux de définir des attributs séparément pour `<input>` un qui prend en charge de nombreux paramètres.

Dans l’exemple suivant, le premier `<input>` élément (`id="useIndividualParams"`) utilise des paramètres de composant individuels, tandis que`id="useAttributesDict"`le deuxième `<input>` élément () utilise la projection d’attributs:

```cshtml
<input id="useIndividualParams"
       maxlength="@Maxlength"
       placeholder="@Placeholder"
       required="@Required"
       size="@Size" />

<input id="useAttributesDict"
       @attributes="InputAttributes" />

@code {
    [Parameter]
    private string Maxlength { get; set; } = "10";

    [Parameter]
    private string Placeholder { get; set; } = "Input placeholder text";

    [Parameter]
    private string Required { get; set; } = "required";

    [Parameter]
    private string Size { get; set; } = "50";

    [Parameter]
    private Dictionary<string, object> InputAttributes { get; set; } =
        new Dictionary<string, object>()
        {
            { "maxlength", "10" },
            { "placeholder", "Input placeholder text" },
            { "required", "true" },
            { "size", "50" }
        };
}
```

Le type du paramètre doit implémenter `IEnumerable<KeyValuePair<string, object>>` avec des clés de chaîne. L' `IReadOnlyDictionary<string, object>` utilisation de est également une option dans ce scénario.

Les éléments `<input>` rendus à l’aide des deux approches sont identiques:

```html
<input id="useIndividualParams"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">

<input id="useAttributesDict"
       maxlength="10"
       placeholder="Input placeholder text"
       required="true"
       size="50">
```

Pour accepter des attributs arbitraires, définissez un paramètre de `[Parameter]` composant à l' `CaptureUnmatchedValues` aide de l' `true`attribut avec la propriété définie sur:

```cshtml
@code {
    [Parameter(CaptureUnmatchedValues = true)]
    private Dictionary<string, object> InputAttributes { get; set; }
}
```

La `CaptureUnmatchedValues` propriété sur `[Parameter]` permet au paramètre de correspondre à tous les attributs qui ne correspondent à aucun autre paramètre. Un composant ne peut définir qu’un seul paramètre `CaptureUnmatchedValues`avec. Le type de propriété utilisé `CaptureUnmatchedValues` avec doit pouvoir être assigné `Dictionary<string, object>` à partir de avec des clés de chaîne. `IEnumerable<KeyValuePair<string, object>>`ou `IReadOnlyDictionary<string, object>` sont également des options dans ce scénario.

## <a name="data-binding"></a>Liaison de données

La liaison de données aux composants et aux éléments DOM s’effectue [@bind](xref:mvc/views/razor#bind) à l’aide de l’attribut. L’exemple suivant lie le `_italicsCheck` champ à l’état activé de la case à cocher:

```cshtml
<input type="checkbox" class="form-check-input" id="italicsCheck" 
    @bind="_italicsCheck" />
```

Lorsque la case à cocher est activée et désactivée, la valeur de `true` la `false`propriété est mise à jour à et, respectivement.

La case à cocher est mise à jour dans l’interface utilisateur uniquement lorsque le composant est rendu, et non en réponse à la modification de la valeur de la propriété. Étant donné que les composants s’affichent après l’exécution du code du gestionnaire d’événements, les mises à jour de propriétés sont généralement reflétées dans l’interface utilisateur immédiatement.

L' `@bind` utilisation de `CurrentValue` avec une`<input @bind="CurrentValue" />`propriété () équivaut essentiellement à ce qui suit:

```cshtml
<input value="@CurrentValue"
    @onchange="@((UIChangeEventArgs __e) => CurrentValue = __e.Value)" />
```

Lors du rendu du composant, le `value` de l’élément d’entrée provient de `CurrentValue` la propriété. Lorsque l’utilisateur tape dans la zone de texte, `onchange` l’événement est déclenché et `CurrentValue` la propriété est définie sur la valeur modifiée. En réalité, la génération de code est un peu plus complexe `@bind` , car gère quelques cas où des conversions de type sont effectuées. En principe, `@bind` associe la valeur actuelle d’une expression à `value` un attribut et gère les modifications à l’aide du gestionnaire inscrit.

En plus de gérer `onchange` les événements `@bind` avec la syntaxe, une propriété ou un champ peut être lié à l’aide [@bind-value](xref:mvc/views/razor#bind) d’autres événements `event` en spécifiant un attribut avec un paramètre ([@bind-value:event](xref:mvc/views/razor#bind)). L’exemple suivant lie la `CurrentValue` propriété de l' `oninput` événement:

```cshtml
<input @bind-value="CurrentValue" @bind-value:event="oninput" />
```

Contrairement `onchange`à, qui se déclenche lorsque l’élément perd `oninput` le focus, se déclenche lorsque la valeur de la zone de texte change.

**Chaînes de format**

La liaison de données <xref:System.DateTime> fonctionne avec les [@bind:format](xref:mvc/views/razor#bind)chaînes de format à l’aide de. D’autres expressions de mise en forme, telles que les formats de devise ou de nombre, ne sont pas disponibles pour l’instant.

```cshtml
<input @bind="StartDate" @bind:format="yyyy-MM-dd" />

@code {
    [Parameter]
    private DateTime StartDate { get; set; } = new DateTime(2020, 1, 1);
}
```

L' `@bind:format` attribut spécifie le format de date à appliquer `value` au de `<input>` l’élément. Le format est également utilisé pour analyser la valeur lorsqu’un `onchange` événement se produit.

**Paramètres du composant**

La liaison reconnaît les paramètres du `@bind-{property}` composant, où peut lier une valeur de propriété à plusieurs composants.

Le composant enfant suivant (`ChildComponent`) a un `Year` paramètre de composant `YearChanged` et un rappel:

```cshtml
<h2>Child Component</h2>

<p>Year: @Year</p>

@code {
    [Parameter]
    private int Year { get; set; }

    [Parameter]
    private EventCallback<int> YearChanged { get; set; }
}
```

`EventCallback<T>`est expliqué dans la section [EventCallback suivante](#eventcallback) .

Le composant parent suivant utilise `ChildComponent` et lie le `ParentYear` paramètre `Year` du parent au paramètre sur le composant enfant:

```cshtml
@page "/ParentComponent"

<h1>Parent Component</h1>

<p>ParentYear: @ParentYear</p>

<ChildComponent @bind-Year="ParentYear" />

<button class="btn btn-primary" @onclick="ChangeTheYear">
    Change Year to 1986
</button>

@code {
    [Parameter]
    private int ParentYear { get; set; } = 1978;

    private void ChangeTheYear()
    {
        ParentYear = 1986;
    }
}
```

Le chargement `ParentComponent` de génère le balisage suivant:

```html
<h1>Parent Component</h1>

<p>ParentYear: 1978</p>

<h2>Child Component</h2>

<p>Year: 1978</p>
```

Si la valeur de la `ParentYear` propriété est modifiée en sélectionnant le bouton dans `ParentComponent`le `ChildComponent` , `Year` la propriété de est mise à jour. La nouvelle valeur de `Year` est rendue dans l’interface utilisateur lorsque `ParentComponent` le est restitué à nouveau:

```html
<h1>Parent Component</h1>

<p>ParentYear: 1986</p>

<h2>Child Component</h2>

<p>Year: 1986</p>
```

Le `Year` paramètre peut être lié, car il a un `YearChanged` événement auxiliaire qui `Year` correspond au type du paramètre.

Par Convention, `<ChildComponent @bind-Year="ParentYear" />` est essentiellement équivalent à l’écriture:

```cshtml
<ChildComponent @bind-Year="ParentYear" @bind-Year:event="YearChanged" />
```

En général, une propriété peut être liée à un gestionnaire d’événements correspondant `@bind-property:event` à l’aide de l’attribut. Par exemple, la propriété `MyProp` peut être liée à `MyEventHandler` à l’aide des deux attributs suivants:

```cshtml
<MyComponent @bind-MyProp="MyValue" @bind-MyProp:event="MyEventHandler" />
```

## <a name="event-handling"></a>Gestion des événements

Les composants Razor fournissent des fonctionnalités de gestion des événements. Pour un attribut d’élément HTML `on{event}` nommé (par exemple `onclick` , `onsubmit`et) avec une valeur typée de type délégué, les composants Razor traitent la valeur de l’attribut en tant que gestionnaire d’événements. Le nom de l’attribut est toujours mis en forme [ @on{Event}](xref:mvc/views/razor#onevent).

Le code suivant appelle la `UpdateHeading` méthode lorsque le bouton est sélectionné dans l’interface utilisateur:

```cshtml
<button class="btn btn-primary" @onclick="UpdateHeading">
    Update heading
</button>

@code {
    private void UpdateHeading(UIMouseEventArgs e)
    {
        ...
    }
}
```

Le code suivant appelle la `CheckChanged` méthode lorsque la case à cocher est modifiée dans l’interface utilisateur:

```cshtml
<input type="checkbox" class="form-check-input" @onchange="CheckChanged" />

@code {
    private void CheckChanged()
    {
        ...
    }
}
```

Les gestionnaires d’événements peuvent également être asynchrones et <xref:System.Threading.Tasks.Task>retourner un. Il n’est pas nécessaire d’appeler `StateHasChanged()`manuellement. Les exceptions sont journalisées lorsqu’elles se produisent.

Dans l’exemple suivant, `UpdateHeading` est appelé de façon asynchrone quand le bouton est sélectionné:

```cshtml
<button class="btn btn-primary" @onclick="UpdateHeading">
    Update heading
</button>

@code {
    private async Task UpdateHeading(UIMouseEventArgs e)
    {
        ...
    }
}
```

### <a name="event-argument-types"></a>Types d’arguments d’événement

Pour certains événements, les types d’arguments d’événement sont autorisés. Si l’accès à l’un de ces types d’événements n’est pas nécessaire, il n’est pas obligatoire dans l’appel de la méthode.

Les [UIEventArgs](https://github.com/aspnet/AspNetCore/blob/release/3.0-preview8/src/Components/Components/src/UIEventArgs.cs) prises en charge sont indiquées dans le tableau suivant.

| Événement | Classe |
| ----- | ----- |
| Presse-papiers | `UIClipboardEventArgs` |
| Déplacez  | `UIDragEventArgs`est utilisé pour contenir les données glissées pendant une opération de glisser-déplacer et peut contenir un ou `UIDataTransferItem`plusieurs. &ndash; `DataTransfer` `UIDataTransferItem`représente un élément de données de glissement. |
| Error | `UIErrorEventArgs` |
| Focus | `UIFocusEventArgs`N’inclut pas la prise en charge de `relatedTarget`. &ndash; |
| Modification de`<input>` | `UIChangeEventArgs` |
| Clavier | `UIKeyboardEventArgs` |
| Souris | `UIMouseEventArgs` |
| Pointeur de la souris | `UIPointerEventArgs` |
| Roulette de la souris | `UIWheelEventArgs` |
| Progression | `UIProgressEventArgs` |
| Entrées tactiles | `UITouchEventArgs`&ndash; représenteunpointdecontactunique`UITouchPoint` sur un appareil tactile. |

Pour plus d’informations sur les propriétés et le comportement de gestion des événements des événements du tableau précédent, consultez [classes EventArgs dans la source de référence](https://github.com/aspnet/AspNetCore/tree/release/3.0-preview8/src/Components/Web/src).

### <a name="lambda-expressions"></a>Expressions lambda

Les expressions lambda peuvent également être utilisées:

```cshtml
<button @onclick="@(e => Console.WriteLine("Hello, world!"))">Say hello</button>
```

Il est souvent pratique de se rapprocher de valeurs supplémentaires, par exemple lors de l’itération sur un ensemble d’éléments. L’exemple suivant crée trois boutons, chacun d’entre eux `UpdateHeading` qui passe un argument d'`UIMouseEventArgs`événement () et son numéro`buttonNumber`de bouton () lorsqu’ils sont sélectionnés dans l’interface utilisateur:

```cshtml
<h2>@message</h2>

@for (var i = 1; i < 4; i++)
{
    var buttonNumber = i;

    <button class="btn btn-primary"
            @onclick="@(e => UpdateHeading(e, buttonNumber))">
        Button #@i
    </button>
}

@code {
    private string message = "Select a button to learn its position.";

    private void UpdateHeading(UIMouseEventArgs e, int buttonNumber)
    {
        message = $"You selected Button #{buttonNumber} at " +
            $"mouse position: {e.ClientX} X {e.ClientY}.";
    }
}
```

> [!NOTE]
> N’utilisez **pas** la variable de boucle`i`() dans `for` une boucle directement dans une expression lambda. Dans le cas contraire, la même variable est utilisée par `i`toutes les expressions lambda, ce qui signifie que la valeur de est identique dans toutes les expressions lambda. Capturez toujours sa valeur dans une variable locale`buttonNumber` (dans l’exemple précédent), puis utilisez-la.

### <a name="eventcallback"></a>EventCallback suivante

Un scénario courant avec des composants imbriqués est le désir d’exécuter la méthode d’un composant parent lorsqu’un événement de&mdash;composant enfant se produit, `onclick` par exemple, lorsqu’un événement se produit dans l’enfant. Pour exposer des événements entre les composants, `EventCallback`utilisez un. Un composant parent peut affecter une méthode de rappel à un composant `EventCallback`enfant.

L `ChildComponent` 'de l’exemple d’application montre comment un `onclick` gestionnaire de bouton est configuré pour recevoir `EventCallback` un délégué à partir de `ParentComponent`l’exemple de. Le `EventCallback` est typé avec `UIMouseEventArgs`, ce qui est approprié pour `onclick` un événement à partir d’un périphérique:

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/ChildComponent.razor?highlight=5-7,17-18)]

Définit la méthode`ShowMessage` de `EventCallback<T>`l’enfant: `ParentComponent`

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=6,16-19)]

Lorsque le bouton est sélectionné dans le `ChildComponent`:

* La `ParentComponent`méthode `ShowMessage` de est appelée. `messageText`est mis à jour et affiché `ParentComponent`dans le.
* Un appel à `StateHasChanged` n’est pas requis dans la méthode du`ShowMessage`rappel (). `StateHasChanged`est appelé automatiquement pour rerestituer `ParentComponent`le, tout comme les événements enfants déclenchent le rerendu des composants dans les gestionnaires d’événements qui s’exécutent dans l’enfant.

`EventCallback`et `EventCallback<T>` autorisent les délégués asynchrones. `EventCallback<T>`est fortement typé et requiert un type d’argument spécifique. `EventCallback`est faiblement typé et autorise tout type d’argument.

```cshtml
<p><b>@messageText</b></p>

@{ var message = "Default Text"; }

<ChildComponent 
    OnClick="@(async () => { await Task.Yield(); messageText = "Blaze It!"; })" />

@code {
    private string messageText;
}
```

`InvokeAsync` <xref:System.Threading.Tasks.Task>Appelez ou avec`EventCallback<T>` et en attente de: `EventCallback`

```csharp
await callback.InvokeAsync(arg);
```

Utilisez `EventCallback` et`EventCallback<T>` pour la gestion des événements et les paramètres de composant de liaison.

Préférez le fortement typé `EventCallback<T>`. `EventCallback` `EventCallback<T>`fournit un meilleur retour d’erreur aux utilisateurs du composant. Comme pour d’autres gestionnaires d’événements d’interface utilisateur, la spécification du paramètre d’événement est facultative. Utilisez `EventCallback` quand aucune valeur n’est passée au rappel.

## <a name="capture-references-to-components"></a>Capturer des références à des composants

Les références de composant offrent un moyen de référencer une instance de composant afin que vous puissiez émettre des commandes vers `Show` cette `Reset`instance, telles que ou. Pour capturer une référence de composant, ajoutez [@ref](xref:mvc/views/razor#ref) un attribut au composant enfant, puis définissez un champ avec le même nom et le même type que le composant enfant.

```cshtml
<MyLoginDialog @ref="loginDialog" ... />

@code {
    private MyLoginDialog loginDialog;

    private void OnSomething()
    {
        loginDialog.Show();
    }
}
```

Lors du rendu du composant, `loginDialog` le champ est rempli avec l’instance du `MyLoginDialog` composant enfant. Vous pouvez ensuite appeler des méthodes .NET sur l’instance du composant.

> [!IMPORTANT]
> La `loginDialog` variable est remplie uniquement après le rendu du composant et sa sortie comprend l' `MyLoginDialog` élément. Jusqu’à ce stade, il n’y a rien à référencer. Pour manipuler des références de composants après la fin du rendu du composant `OnAfterRenderAsync` , `OnAfterRender` utilisez les méthodes ou.

Bien que la capture de références de composant utilise une syntaxe similaire pour [capturer des références d’élément](xref:blazor/javascript-interop#capture-references-to-elements), il ne s’agit pas d’une fonctionnalité [JavaScript Interop](xref:blazor/javascript-interop) . Les références de composant ne sont pas&mdash;transmises au code JavaScript et ne sont utilisées que dans le code .net.

> [!NOTE]
> N’utilisez **pas** de références de composant pour muter l’état des composants enfants. Utilisez plutôt des paramètres déclaratifs normaux pour passer des données aux composants enfants. L’utilisation de paramètres déclaratifs normaux entraîne le rerendu automatique des composants enfants.

## <a name="use-key-to-control-the-preservation-of-elements-and-components"></a>Utiliser \@la clé pour contrôler la conservation des éléments et des composants

Lors du rendu d’une liste d’éléments ou de composants, et que les éléments ou composants changent par la suite, l’algorithme de différenciation de éblouissant doit décider quel élément ou composant précédent peut être conservé et comment les objets de modèle doivent être mappés à ces éléments. Normalement, ce processus est automatique et peut être ignoré, mais dans certains cas, il peut être utile de contrôler le processus.

Prenons l'exemple suivant :

```csharp
@foreach (var person in People)
{
    <DetailsEditor Details="@person.Details" />
}

@code {
    [Parameter]
    private IEnumerable<Person> People { get; set; }
}
```

Le contenu de la `People` collection peut changer avec des entrées insérées, supprimées ou réorganisées. Lors du rerendu du composant, le `<DetailsEditor>` composant peut changer pour recevoir des `Details` valeurs de paramètre différentes. Cela peut entraîner un rerendu plus complexe que prévu. Dans certains cas, le rerendu peut entraîner des différences de comportement visibles, telles que le focus de l’élément perdu.

Le processus de mappage peut être contrôlé à `@key` l’aide de l’attribut directive. `@key`force l’algorithme de comparaison à garantir la préservation des éléments ou des composants en fonction de la valeur de la clé:

```csharp
@foreach (var person in People)
{
    <DetailsEditor @key="@person" Details="@person.Details" />
}

@code {
    [Parameter]
    private IEnumerable<Person> People { get; set; }
}
```

Lorsque la `People` collection est modifiée, l’algorithme de comparaison conserve l’Association `<DetailsEditor>` entre les `person` instances et les instances:

* Si un `Person` est supprimé de la `People` liste, seule l’instance `<DetailsEditor>` correspondante est supprimée de l’interface utilisateur. Les autres instances restent inchangées.
* Si un `Person` est inséré à une position dans la liste, une nouvelle `<DetailsEditor>` instance est insérée à cette position correspondante. Les autres instances restent inchangées.
* Si `Person` les entrées sont réordonnées, les `<DetailsEditor>` instances correspondantes sont conservées et réordonnées dans l’interface utilisateur.

Dans certains scénarios, l’utilisation `@key` de réduit la complexité du rerendu et évite les problèmes potentiels liés aux parties avec état de la modification du modèle DOM, telles que la position du focus.

> [!IMPORTANT]
> Les clés sont locales pour chaque élément ou composant conteneur. Les clés ne sont pas comparées globalement dans le document.

### <a name="when-to-use-key"></a>Quand utiliser \@la clé

En règle générale, il est judicieux `@key` d’utiliser chaque fois qu’une liste est rendue (par `@foreach` exemple, dans un bloc) et qu’une `@key`valeur appropriée existe pour définir.

Vous pouvez également utiliser `@key` pour empêcher éblouissant de conserver un élément ou une sous-arborescence de composants lorsqu’un objet change:

```cshtml
<div @key="@currentPerson">
    ... content that depends on @currentPerson ...
</div>
```

En `@currentPerson` cas de modification `@key` , la directive d’attribut force éblouissant à ignorer `<div>` la totalité et ses descendants et à reconstruire la sous-arborescence de l’interface utilisateur avec de nouveaux éléments et composants. Cela peut être utile si vous devez garantir qu’aucun État d’interface utilisateur n’est `@currentPerson` préservé lorsque des modifications sont apportées.

### <a name="when-not-to-use-key"></a>Quand ne pas utiliser \@la clé

Il y a un coût en matière de performances `@key`lors de la comparaison avec. Le coût des performances n’est pas important, `@key` mais spécifiez uniquement si les règles de conservation des éléments ou des composants bénéficient de l’application.

Même si `@key` n’est pas utilisé, éblouissant conserve autant que possible les instances d’élément et de composant enfants. Le seul avantage de l' `@key` utilisation de est de contrôler la *façon dont* les instances de modèle sont mappées aux instances de composant conservées, au lieu de l’algorithme de comparaison qui sélectionne le mappage.

### <a name="what-values-to-use-for-key"></a>Valeurs à utiliser pour \@la clé

En règle générale, il est logique de fournir l’un des types de valeur `@key`suivants pour:

* Instances d’objet de modèle (par exemple `Person` , une instance comme dans l’exemple précédent). Cela garantit la préservation en fonction de l’égalité des références d’objet.
* Identificateurs uniques (par exemple, les valeurs de clé primaire `int`de `string`type, `Guid`ou).

Évitez de fournir une valeur qui peut entrer en conflit de manière inattendue. Si `@key="@someObject.GetHashCode()"` est fourni, des conflits inattendus peuvent se produire parce que les codes de hachage des objets non liés peuvent être identiques. Si les `@key` valeurs en conflit sont demandées dans le même parent `@key` , les valeurs ne sont pas respectées.

## <a name="lifecycle-methods"></a>Méthodes de cycle de vie

`OnInitAsync`et `OnInit` exécuter le code pour initialiser le composant. Pour effectuer une opération asynchrone, utilisez `OnInitAsync` et le `await` mot clé sur l’opération:

```csharp
protected override async Task OnInitAsync()
{
    await ...
}
```

Pour une opération synchrone, utilisez `OnInit`:

```csharp
protected override void OnInit()
{
    ...
}
```

`OnParametersSetAsync`et `OnParametersSet` sont appelées lorsqu’un composant a reçu des paramètres de son parent et que les valeurs sont assignées aux propriétés. Ces méthodes sont exécutées après l’initialisation du composant et chaque fois que le composant est rendu:

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

```csharp
protected override void OnParametersSet()
{
    ...
}
```

`OnAfterRenderAsync`et `OnAfterRender` sont appelées après la fin d’un rendu d’un composant. Les références d’élément et de composant sont remplies à ce stade. Utilisez cette étape pour effectuer des étapes d’initialisation supplémentaires à l’aide du contenu rendu, par exemple l’activation de bibliothèques JavaScript tierces qui opèrent sur les éléments DOM rendus.

```csharp
protected override async Task OnAfterRenderAsync()
{
    await ...
}
```

```csharp
protected override void OnAfterRender()
{
    ...
}
```

### <a name="handle-incomplete-async-actions-at-render"></a>Gérer les actions asynchrones incomplètes au rendu

Les actions asynchrones exécutées dans des événements de cycle de vie peuvent ne pas être terminées avant le rendu du composant. Les objets peuvent `null` être ou être remplis de façon incomplète avec des données pendant l’exécution de la méthode Lifecycle. Fournissez une logique de rendu pour confirmer que les objets sont initialisés. Affichez les éléments d’interface utilisateur d’espace réservé (par exemple, un `null`message de chargement) tandis que les objets sont.

Dans le `FetchData` composant des modèles éblouissant, `OnInitAsync` est remplacé par asynchrone recevoir les données de prévision (`forecasts`). Lorsque `forecasts` a `null`la valeur, un message de chargement est affiché à l’utilisateur. Une fois `Task` la `OnInitAsync` retournée terminée, le composant est rerendu avec l’État mis à jour.

*Pages/FetchData.razor* :

[!code-cshtml[](components/samples_snapshot/3.x/FetchData.razor?highlight=9)]

### <a name="execute-code-before-parameters-are-set"></a>Exécuter du code avant la définition des paramètres

`SetParameters`peut être substitué pour exécuter du code avant que les paramètres soient définis:

```csharp
public override void SetParameters(ParameterCollection parameters)
{
    ...

    base.SetParameters(parameters);
}
```

Si `base.SetParameters` n’est pas appelé, le code personnalisé peut interpréter la valeur des paramètres entrants de la façon requise. Par exemple, il n’est pas nécessaire que les paramètres entrants soient assignés aux propriétés de la classe.

### <a name="suppress-refreshing-of-the-ui"></a>Supprimer l’actualisation de l’interface utilisateur

`ShouldRender`peut être substitué pour supprimer l’actualisation de l’interface utilisateur. Si l’implémentation retourne `true`, l’interface utilisateur est actualisée. Même si `ShouldRender` est substitué, le composant est toujours restitué initialement.

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

## <a name="component-disposal-with-idisposable"></a>Suppression de composant avec IDisposable

Si un composant implémente <xref:System.IDisposable>, la [méthode dispose](/dotnet/standard/garbage-collection/implementing-dispose) est appelée lorsque le composant est supprimé de l’interface utilisateur. Le composant suivant utilise `@implements IDisposable` et la `Dispose` méthode:

```csharp
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

## <a name="routing"></a>Routage

Le routage dans éblouissant est obtenu en fournissant un modèle de routage à chaque composant accessible dans l’application.

Lorsqu’un fichier Razor avec une `@page` directive est compilé, la classe générée reçoit un qui <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> spécifie le modèle de routage. Lors de l’exécution, le routeur recherche les classes de `RouteAttribute` composant avec un et rend le composant qui a un modèle de routage correspondant à l’URL demandée.

Plusieurs modèles de routage peuvent être appliqués à un composant. Le composant suivant répond aux demandes pour `/BlazorRoute` et `/DifferentBlazorRoute`:

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/BlazorRoute.razor?name=snippet_BlazorRoute)]

## <a name="route-parameters"></a>Paramètres d’itinéraire

Les composants peuvent recevoir des paramètres de routage du modèle de routage `@page` fourni dans la directive. Le routeur utilise des paramètres de routage pour remplir les paramètres de composant correspondants.

*Composant de paramètre de routage*:

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/RouteParameter.razor?name=snippet_RouteParameter)]

Les paramètres facultatifs ne sont pas `@page` pris en charge. deux directives sont donc appliquées dans l’exemple ci-dessus. La première permet de naviguer jusqu’au composant sans paramètre. La deuxième `@page` directive prend le `{text}` paramètre d’itinéraire et assigne la valeur à `Text` la propriété.

## <a name="base-class-inheritance-for-a-code-behind-experience"></a>Héritage de la classe de base pour une expérience «code-behind»

Les fichiers de composants associent C# le balisage HTML et le code de traitement dans le même fichier. La `@inherits` directive peut être utilisée pour fournir des applications éblouissantes avec une expérience «code-behind» qui sépare le balisage de composant du code de traitement.

L' [exemple d’application](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) montre comment un composant peut hériter d’une `BlazorRocksBase`classe de base,, pour fournir les propriétés et les méthodes du composant.

*Pages/BlazorRocks. Razor*:

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/BlazorRocks.razor?name=snippet_BlazorRocks)]

*BlazorRocksBase.cs*:

[!code-csharp[](common/samples/3.x/BlazorSample/Pages/BlazorRocksBase.cs)]

La classe de base doit dériver de `ComponentBase`.

## <a name="import-components"></a>Importer des composants

L’espace de noms d’un composant créé avec Razor est basé sur:

* Du `RootNamespace`projet.
* Chemin d’accès de la racine du projet au composant. Par exemple, `ComponentsSample/Pages/Index.razor` se trouve dans l' `ComponentsSample.Pages`espace de noms. Les composants C# suivent les règles de liaison de nom. Dans le cas de *index. Razor*, tous les composants dans le même dossier, les mêmes *pages*et le dossier parent, *ComponentsSample*, sont dans l’étendue.

Les composants définis dans un espace de noms différent peuvent être mis en portée à l’aide de la directive [ \@using](xref:mvc/views/razor#using) de Razor.

Si un autre composant `NavMenu.razor`,, existe dans le `ComponentsSample/Shared/`dossier, le composant peut être utilisé `Index.razor` dans avec l' `@using` instruction suivante:

```cshtml
@using ComponentsSample.Shared

This is the Index page.

<NavMenu></NavMenu>
```

Les composants peuvent également être référencés à l’aide de leurs noms complets, ce qui supprime [ \@](xref:mvc/views/razor#using) la nécessité de la directive using:

```cshtml
This is the Index page.

<ComponentsSample.Shared.NavMenu></ComponentsSample.Shared.NavMenu>
```

> [!NOTE]
> La `global::` qualification n’est pas prise en charge.
>
> L’importation de composants avec `using` des instructions avec alias ( `@using Foo = Bar`par exemple,) n’est pas prise en charge.
>
> Les noms partiellement qualifiés ne sont pas pris en charge. Par exemple, l' `@using ComponentsSample` ajout et `NavMenu.razor` la `<Shared.NavMenu></Shared.NavMenu>` référencement avec ne sont pas pris en charge.

## <a name="conditional-html-element-attributes"></a>Attributs d’éléments HTML conditionnels

Les attributs des éléments HTML sont rendus de manière conditionnelle en fonction de la valeur .NET. Si la valeur est `false` ou `null`, l’attribut n’est pas rendu. Si la valeur est `true`, l’attribut est rendu réduit.

Dans l’exemple suivant, `IsCompleted` détermine si `checked` est rendu dans le balisage de l’élément:

```cshtml
<input type="checkbox" checked="@IsCompleted" />

@code {
    [Parameter]
    private bool IsCompleted { get; set; }
}
```

Si `IsCompleted` est`true`, la case à cocher s’affiche comme suit:

```html
<input type="checkbox" checked />
```

Si `IsCompleted` est`false`, la case à cocher s’affiche comme suit:

```html
<input type="checkbox" />
```

Pour plus d'informations, consultez <xref:mvc/views/razor>.

## <a name="raw-html"></a>HTML brut

Les chaînes sont normalement rendues à l’aide de nœuds de texte DOM, ce qui signifie que tout balisage qu’elles peuvent contenir est ignorée et traitée comme du texte littéral. Pour afficher le code HTML brut, encapsulez le `MarkupString` contenu HTML dans une valeur. La valeur est analysée au format HTML ou SVG et est insérée dans le DOM.

> [!WARNING]
> Le rendu de code HTML brut construit à partir de n’importe quelle source non approuvée constitue un risque pour la **sécurité** et doit être évité!

L’exemple suivant illustre l’utilisation `MarkupString` du type pour ajouter un bloc de contenu HTML statique à la sortie rendue d’un composant:

```html
@((MarkupString)myMarkup)

@code {
    private string myMarkup = 
        "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```

## <a name="templated-components"></a>Composants basés sur un modèle

Les composants basés sur un modèle sont des composants qui acceptent un ou plusieurs modèles d’interface utilisateur en tant que paramètres, qui peuvent ensuite être utilisés dans le cadre de la logique de rendu du composant. Les composants basés sur un modèle vous permettent de créer des composants de niveau supérieur qui sont plus réutilisables que les composants normaux. Voici quelques exemples:

* Composant de table qui permet à un utilisateur de spécifier des modèles pour l’en-tête, les lignes et le pied de page de la table.
* Composant de liste qui permet à un utilisateur de spécifier un modèle pour afficher des éléments dans une liste.

### <a name="template-parameters"></a>Paramètres de modèle

Un composant basé sur un modèle est défini en spécifiant un ou plusieurs paramètres `RenderFragment` de `RenderFragment<T>`composant de type ou. Un fragment de rendu représente un segment de l’interface utilisateur à restituer. `RenderFragment<T>`prend un paramètre de type qui peut être spécifié lors de l’appel du fragment de rendu.

`TableTemplate`-

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/TableTemplate.razor)]

Lorsque vous utilisez un composant basé sur un modèle, les paramètres de modèle peuvent être spécifiés à l’aide d’éléments enfants`TableHeader` qui `RowTemplate` correspondent aux noms des paramètres (et dans l’exemple suivant):

```cshtml
<TableTemplate Items="@pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@context.PetId</td>
        <td>@context.Name</td>
    </RowTemplate>
</TableTemplate>
```

### <a name="template-context-parameters"></a>Paramètres de contexte de modèle

Les arguments de composant `RenderFragment<T>` de type passé comme éléments ont un paramètre `context` implicite nommé (par exemple, à partir `@context.PetId`de l’exemple de code précédent,), mais vous `Context` pouvez modifier le nom de paramètre à l’aide de l’attribut sur l’enfant. appartient. Dans l’exemple suivant, l' `RowTemplate` attribut de `Context` l’élément spécifie le `pet` paramètre:

```cshtml
<TableTemplate Items="@pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate Context="pet">
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

Vous pouvez également spécifier l' `Context` attribut sur l’élément de composant. L’attribut `Context` spécifié s’applique à tous les paramètres de modèle spécifiés. Cela peut être utile lorsque vous souhaitez spécifier le nom du paramètre de contenu pour le contenu enfant implicite (sans élément enfant d’encapsulation). Dans l’exemple suivant, l' `Context` attribut apparaît sur l' `TableTemplate` élément et s’applique à tous les paramètres de modèle:

```cshtml
<TableTemplate Items="@pets" Context="pet">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

### <a name="generic-typed-components"></a>Composants génériques

Les composants basés sur un modèle sont souvent typés de façon générique. Par exemple, un composant `ListViewTemplate` générique peut être utilisé pour restituer `IEnumerable<T>` des valeurs. Pour définir un composant générique, utilisez la `@typeparam` directive pour spécifier les paramètres de type:

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/ListViewTemplate.razor)]

Lorsque vous utilisez des composants génériques, le paramètre de type est déduit si possible:

```cshtml
<ListViewTemplate Items="@pets">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

Sinon, le paramètre de type doit être spécifié explicitement à l’aide d’un attribut qui correspond au nom du paramètre de type. Dans l’exemple suivant, `TItem="Pet"` spécifie le type:

```cshtml
<ListViewTemplate Items="@pets" TItem="Pet">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

## <a name="cascading-values-and-parameters"></a>Valeurs et paramètres en cascade

Dans certains scénarios, il est peu commode de transmettre des données d’un composant ancêtre à un composant descendant à l’aide de [paramètres de composant](#component-parameters), en particulier lorsqu’il existe plusieurs couches de composant. Les valeurs et paramètres en cascade résolvent ce problème en fournissant un moyen pratique pour un composant ancêtre de fournir une valeur à tous ses composants descendants. Les valeurs et les paramètres en cascade fournissent également une approche pour les composants à coordonner.

### <a name="theme-example"></a>Exemple de thème

Dans l’exemple suivant tiré de l’exemple d’application `ThemeInfo` , la classe spécifie les informations de thème pour descendre dans la hiérarchie des composants, de sorte que tous les boutons d’une partie donnée de l’application partagent le même style.

*UIThemeClasses/themeinfo. cs*:

```csharp
public class ThemeInfo
{
    public string ButtonClass { get; set; }
}
```

Un composant ancêtre peut fournir une valeur en cascade à l’aide du composant de valeur en cascade. Le `CascadingValue` composant encapsule une sous-arborescence de la hiérarchie des composants et fournit une valeur unique à tous les composants de cette sous-arborescence.

Par exemple, l’exemple d’application spécifie les`ThemeInfo`informations de thème () dans l’une des dispositions de l’application en tant que paramètre en cascade pour tous les composants qui composent le corps de la disposition de la `@Body` propriété. `ButtonClass`la valeur `btn-success` est affectée à dans le composant Layout. Tout composant descendant peut consommer cette propriété par le `ThemeInfo` biais de l’objet en cascade.

`CascadingValuesParametersLayout`-

```cshtml
@inherits LayoutComponentBase
@using BlazorSample.UIThemeClasses

<div class="container-fluid">
    <div class="row">
        <div class="col-sm-3">
            <NavMenu />
        </div>
        <div class="col-sm-9">
            <CascadingValue Value="@theme">
                <div class="content px-4">
                    @Body
                </div>
            </CascadingValue>
        </div>
    </div>
</div>

@code {
    private ThemeInfo theme = new ThemeInfo { ButtonClass = "btn-success" };
}
```

Pour utiliser des valeurs en cascade, les composants déclarent des paramètres en cascade `[CascadingParameter]` à l’aide de l’attribut ou d’une valeur de nom de chaîne:

```cshtml
<CascadingValue Value=@PermInfo Name="UserPermissions">...</CascadingValue>

[CascadingParameter(Name = "UserPermissions")]
private PermInfo Permissions { get; set; }
```

La liaison avec une valeur de nom de chaîne est pertinente si vous avez plusieurs valeurs en cascade du même type et que vous devez les différencier dans la même sous-arborescence.

Les valeurs en cascade sont liées aux paramètres en cascade par type.

Dans l’exemple d’application, `CascadingValuesParametersTheme` le composant lie la `ThemeInfo` valeur en cascade à un paramètre en cascade. Le paramètre est utilisé pour définir la classe CSS pour l’un des boutons affichés par le composant.

`CascadingValuesParametersTheme`-

```cshtml
@page "/cascadingvaluesparameterstheme"
@layout CascadingValuesParametersLayout
@using BlazorSample.UIThemeClasses

<h1>Cascading Values & Parameters</h1>

<p>Current count: @currentCount</p>

<p>
    <button class="btn" @onclick="IncrementCount">
        Increment Counter (Unthemed)
    </button>
</p>

<p>
    <button class="btn @ThemeInfo.ButtonClass" @onclick="IncrementCount">
        Increment Counter (Themed)
    </button>
</p>

@code {
    private int currentCount = 0;

    [CascadingParameter]
    protected ThemeInfo ThemeInfo { get; set; }

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

### <a name="tabset-example"></a>Exemple TabSet

Les paramètres en cascade permettent également aux composants de collaborer au sein de la hiérarchie des composants. Par exemple, considérez l’exemple *TabSet* suivant dans l’exemple d’application.

L’exemple d’application possède `ITab` une interface qui implémente les onglets suivants:

[!code-cs[](common/samples/3.x/BlazorSample/UIInterfaces/ITab.cs)]

Le `CascadingValuesParametersTabSet` composant utilise le `TabSet` composant, qui contient plusieurs `Tab` composants:

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/CascadingValuesParametersTabSet.razor?name=snippet_TabSet)]

Les composants `Tab` enfants ne sont pas explicitement passés comme paramètres `TabSet`à. Au lieu de cela `Tab` , les composants enfants font partie du contenu enfant `TabSet`de. Toutefois, le `TabSet` doit toujours connaître chaque `Tab` composant pour pouvoir afficher les en-têtes et l’onglet actif. Pour activer cette coordination sans nécessiter de code supplémentaire `TabSet` , le composant *peut se présenter comme une valeur en cascade* qui est ensuite récupérée par les `Tab` composants descendants.

`TabSet`-

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/TabSet.razor)]

Les composants `Tab` descendants capturent `TabSet` le contenant comme paramètre en cascade, de `Tab` sorte que les composants s' `TabSet` ajoutent eux-mêmes à la et à la coordonnée sur laquelle l’onglet est actif.

`Tab`-

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/Tab.razor)]

## <a name="razor-templates"></a>Modèles Razor

Les fragments de rendu peuvent être définis à l’aide de la syntaxe de modèle Razor. Les modèles Razor sont un moyen de définir un extrait de code d’interface utilisateur et de supposer le format suivant:

```cshtml
@<{HTML tag}>...</{HTML tag}>
```

L’exemple suivant montre comment spécifier `RenderFragment` des valeurs et `RenderFragment<T>` et restituer des modèles directement dans un composant. Les fragments de rendu peuvent également être passés comme arguments à des [composants basés](#templated-components)sur un modèle.

```cshtml
@timeTemplate

@petTemplate(new Pet { Name = "Rex" })

@code {
    private RenderFragment timeTemplate = @<p>The time is @DateTime.Now.</p>;
    private RenderFragment<Pet> petTemplate = 
        (pet) => @<p>Your pet's name is @pet.Name.</p>;

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

Sortie rendue du code précédent:

```html
<p>The time is 10/04/2018 01:26:52.</p>

<p>Your pet's name is Rex.</p>
```

## <a name="manual-rendertreebuilder-logic"></a>Logique RenderTreeBuilder manuelle

`Microsoft.AspNetCore.Components.RenderTree`fournit des méthodes pour manipuler des composants et des éléments, y compris la C# génération manuelle de composants dans le code.

> [!NOTE]
> L’utilisation `RenderTreeBuilder` de pour créer des composants est un scénario avancé. Un composant mal formé (par exemple, une balise de balisage non fermée) peut entraîner un comportement indéfini.

Prenons le composant `PetDetails` suivant, qui peut être intégré manuellement à un autre composant:

```cshtml
<h2>Pet Details Component</h2>

<p>@PetDetailsQuote</p>

@code
{
    [Parameter]
    private string PetDetailsQuote { get; set; }
}
```

Dans l’exemple suivant, la boucle de la `CreateComponent` méthode génère trois `PetDetails` composants. Lors de `RenderTreeBuilder` l’appel de méthodes pour créer`OpenComponent` les `AddAttribute`composants (et), les numéros de séquence sont des numéros de ligne de code source. L’algorithme de différence éblouissant s’appuie sur les numéros de séquence correspondant à des lignes de code distinctes, et non sur des appels d’appel distincts. Lors de la création d' `RenderTreeBuilder` un composant avec des méthodes, coder en dur les arguments pour les numéros de séquence. **L’utilisation d’un calcul ou d’un compteur pour générer le numéro de séquence peut entraîner des performances médiocres.** Pour plus d’informations, consultez la section [numéros de séquence liés à des numéros de ligne de code et non à un ordre d’exécution](#sequence-numbers-relate-to-code-line-numbers-and-not-execution-order) .

`BuiltContent`-

```cshtml
@page "/BuiltContent"

<h1>Build a component</h1>

@CustomRender

<button type="button" @onclick="RenderComponent">
    Create three Pet Details components
</button>

@code {
    private RenderFragment CustomRender { get; set; }
    
    private RenderFragment CreateComponent() => builder =>
    {
        for (var i = 0; i < 3; i++) 
        {
            builder.OpenComponent(0, typeof(PetDetails));
            builder.AddAttribute(1, "PetDetailsQuote", "Someone's best friend!");
            builder.CloseComponent();
        }
    };    
    
    private void RenderComponent()
    {
        CustomRender = CreateComponent();
    }
}
```

### <a name="sequence-numbers-relate-to-code-line-numbers-and-not-execution-order"></a>Les numéros de séquence sont liés aux numéros de ligne de code et non à l’ordre d’exécution

`.razor` Les fichiers éblouissants sont toujours compilés. C’est un avantage considérable pour, `.razor` car l’étape de compilation peut être utilisée pour injecter des informations qui améliorent les performances de l’application au moment de l’exécution.

Un exemple clé de ces améliorations concerne les *numéros de séquence*. Les numéros de séquence indiquent au runtime que les sorties proviennent de lignes de code distinctes et ordonnées. Le runtime utilise ces informations pour générer des différences d’arborescence efficaces en temps linéaire, ce qui est beaucoup plus rapide qu’un algorithme de comparaison d’arborescence générale.

Considérons le fichier `.razor` simple suivant:

```cshtml
@if (someFlag)
{
    <text>First</text>
}

Second
```

Le code précédent se compile comme suit:

```csharp
if (someFlag)
{
    builder.AddContent(0, "First");
}

builder.AddContent(1, "Second");
```

Lorsque le code s’exécute pour la première fois, si `someFlag` est `true`, le générateur reçoit:

| Séquence | Type      | Données   |
| :------: | --------- | :----: |
| 0        | Nœud de texte | Première  |
| 1        | Nœud de texte | Seconde |

Imaginez que `someFlag` devient `false`et le balisage est de nouveau restitué. Cette fois-ci, le générateur reçoit:

| Séquence | Type       | Données   |
| :------: | ---------- | :----: |
| 1        | Nœud de texte  | Seconde |

Quand le runtime effectue une comparaison, il constate que l’élément au niveau `0` de la séquence a été supprimé. il génère donc le *script d’édition*trivial suivant:

* Supprimez le premier nœud de texte.

#### <a name="what-goes-wrong-if-you-generate-sequence-numbers-programmatically"></a>Qu’est-ce qui se passe si vous générez des numéros séquentiels par programmation

Imaginez plutôt que vous avez écrit la logique de générateur d’arborescence de rendu suivante:

```csharp
var seq = 0;

if (someFlag)
{
    builder.AddContent(seq++, "First");
}

builder.AddContent(seq++, "Second");
```

La première sortie est désormais:

| Séquence | Type      | Données   |
| :------: | --------- | :----: |
| 0        | Nœud de texte | Première  |
| 1        | Nœud de texte | Seconde |

Ce résultat est identique au cas précédent, donc aucun problème négatif n’existe. `someFlag`se `false` trouve sur le deuxième rendu et la sortie est:

| Séquence | Type      | Données   |
| :------: | --------- | ------ |
| 0        | Nœud de texte | Seconde |

Cette fois-ci, l’algorithme diff constate que *deux* modifications ont été apportées, et l’algorithme génère le script Edit suivant:

* Remplacez la valeur du premier nœud de texte par `Second`.
* Supprimez le deuxième nœud de texte.

La génération des numéros de séquence a perdu toutes les informations utiles sur `if/else` l’emplacement où les branches et les boucles étaient présentes dans le code d’origine. Il en résulte une comparaison de **deux fois plus longtemps** qu’auparavant.

Il s’agit d’un exemple trivial. Dans des cas plus réalistes avec des structures complexes et profondément imbriquées, et surtout avec des boucles, le coût des performances est plus grave. Au lieu d’identifier immédiatement les blocs de boucle ou les branches qui ont été insérées ou supprimées, l’algorithme diff doit recurse profondément dans les arborescences de rendu et générer des scripts de modification beaucoup plus longs, car il est mal informé de la façon dont les anciennes et les nouvelles structures sont liées les unes aux autres.

#### <a name="guidance-and-conclusions"></a>Conseils et conclusions

* Les performances de l’application sont affectées si les numéros séquentiels sont générés dynamiquement.
* L’infrastructure ne peut pas créer ses propres numéros de séquence automatiquement au moment de l’exécution, car les informations nécessaires n’existent pas, sauf si elles sont capturées au moment de la compilation.
* N’écrivez pas de longs blocs de logique `RenderTreeBuilder` implémentée manuellement. Préférer `.razor` les fichiers et permettre au compilateur de gérer les numéros de séquence.
* Si les numéros de séquence sont codés en dur, l’algorithme diff exige uniquement que les numéros de séquence augmentent dans la valeur. La valeur initiale et les écarts ne sont pas pertinents. Une option légitime consiste à utiliser le numéro de ligne de code comme numéro de séquence, ou à commencer à partir de zéro et à augmenter par des ou des centaines (ou un intervalle de préférence). 
* Éblouissant utilise des numéros de séquence, tandis que d’autres infrastructures d’interface utilisateur de comparaison d’arborescence ne les utilisent pas. La comparaison est beaucoup plus rapide lorsque les numéros de séquence sont utilisés, et éblouissant présente l’avantage d’une étape de compilation qui traite automatiquement les numéros séquentiels `.razor` pour les développeurs qui créent des fichiers.