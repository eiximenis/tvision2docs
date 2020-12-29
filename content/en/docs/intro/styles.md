---
title: "Styling the application"
linkTitle: "Styling the application"
weight: 8
description: >
  How to create and use styles
---

TVision2 supports the concept of _styles_ to allow you have all color definitions in a single place. To use styles you need to include the package `Tvision2.Styles` and enable them in your `Program.cs` using the `AddStyles` extension method:

```csharp
 builder.UseTvision2(setup => {
        setup.UseDotNetConsoleDriver();
        setup.AddStyles(AddApplicationStyles);      // AddApplicationStyles is a Action<IStyleBuilder>
        // Other configuration
    }
}
```

## Skins and Styles

A _Skin_ is a collection of _Styles_ and a _Style_ is a collection of entries where each entry contains, basically, the foreground and the background colors. You define the application styles at the startup using the `AddStyles` method. This method receives an `Action<ISkinManagerBuilder>` as a parameter (in our code sample was the `AddApplicationStyles` method):

```csharp
private static void AddApplicationStyles(ISkinManagerBuilder sb)
{
    sb.AddDefaultSkin(skin =>  {
        skin.AddBaseStyle(bs => bs.Default().DesiredStandard(sd => sd.UseBackground(TvColor.Red).UseForeground(TvColor.Black)));
        skin.AddStyle("text", bs => bs.Default().DesiredStandard(sd => sd.UseBackground(TvColor.Blue).UseForeground(TvColor.Yellow)));
    });
}
```

Here we are creating one skin (the default skin) with two styles:

1. The "base" style (which will be used if the requested style is not found). This style has its `Standard` entry set to black over red.
2. A style called "text" with its `Standard` entry set to yellow over blue.

> At a runtime, each style is a `IStyle` object and each entry is a `StyleEntry` object.

## The entries of a Style

Each style can have any number of entries, but there are four predefined entries, called `Standard`, `Active`, `Alternate` and `AlternateActive`. You can get those entries from a style using the properties of the same name. However, you can use the indexer to access to any other entry with a specific name.

For example, if `style` is a `IStyle` instance you can use:

```csharp
var std = style.Standard;           // std is a StyleEntry
var ent = style["custom_name"];     // ent is a StyleEntry
```

To define more than one entry in the same style, use more than one `DesiredXXXX` calls in the `AddStyle` method:

```csharp
sb.AddDefaultSkin(skin =>  {
    skin.AddStyle("text", bs => bs.Default()
        .DesiredStandard(sd => sd.UseBackground(TvColor.Blue).UseForeground(TvColor.Yellow))
        .DesiredAlternate(sd => sd.UseBackground(TvColor.Black).UseForeground(TvColor.White))
        .Desired("custom_name", sd => sd.UseBackground(TvColor.Yellow).UseForeground(TvColor.Black))
    );
});
```

The `text` style contains three entries:

* The `Standard` one that is yellow over blue (use the `Standard` property to get this entry)
* The `Alternate` that is white over black (use the `Alternate` property to get this entry)
* One entry called `custom_name` which is black over yellow (use the indexer to get this entry (`["custom_name"]`))

## Using Styles

You need to use the styles in the drawers of your components. The package `Tvision2.Styles` add some extension methods to the `RenderContext` class to allow you use styles easily.

The `Styled` extension method, converts the `RenderContext` to a `StyledRenderContext` which applies automatically the specified style. Following code would be inside a drawer:

```csharp
ctx.Styled("text").DrawStringAt(ctx.State.PadRight(ctx.Viewport.Bounds.Cols), TvPoint.Zero);
```

Note that the method `DrawStringAt` **do not receive any `CharacterAttribute` or `TvColorPair` parameters**. This is because the result of `ctx.Styled("text")` is a `StyledRenderContext` which have the `text` style applied. By default the `StyleRenderContext` uses the value of `Standard` entry to render its contents, but there are overloads to allow you specify which entry of the style should be used.

> For performance reasons, avoid calling `Styled` most than one in a render. If needed store the result in a intermediate variable.

For more complex scenarios you can get the `ISkinManager` object directly, using the `GetSkinManager` extension method of the `RenderContext`. Once you have the `ISkinManager` you can use it to get any entry of any style.

The `ISkinManager` object provides following properties and methods:

* `CurrentSkin`: Returns the current skin (`ISkin`). By default the current skin is the default skin.
* `DefaultSkin`: Returns the default skin (`ISkin`). The default skin is the skin defined using `AddDefaultSkin()`. If a default skin has not been defined, then `DefaultSkin` defaults to the first skin defined.
* `GetSkin(name)`: Return the skin (`ISkin`) with the specified `name`.
* `ChangeCurrentSkin(name)`: Change the current skin to the specified one.
* `SkinNames`: Return all skins names.

The `ISkin` interface just offers two properties:

* `DefaultStyle`: Returns the default style (`IStyle`).
* The _indexer_ with a string parameter: Returns the style (`IStyle`) with the specified name or the default style if not found.

Finally the `IStyle` is the object that contains the 4 `StyleEntry` objects and a indexer to retrieve any additional entry:

```csharp
public interface IStyle
{
    StyleEntry Standard { get; }
    StyleEntry Active { get; }
    StyleEntry Alternate { get; }
    StyleEntry AlternateActive { get; }
    StyleEntry this[string name] { get; }
}
```

So, you can use `ISkinManager` in a way like that (`ctx` is a `RenderContext`) to get the alternate colors definition of the style named `text` from the current used skin:

```csharp
var alternateColors = ctx.GetSkinManager().CurrentSkin["text"].Alternate;
```

## Defining styles

As mentioned before you define your application styles using the `AddStyles` method, and using the `ISkinManagerBuilder` object:

```csharp
builder.UseTvision2(setup => {
    setup.AddStyles(sm => {
        sm.AddDefaultSkin(....);            // Default skin definition (optional)
        sm.AddSkin(...);                    // Additional skins definitions
    });
}
```

Both methods (`AddDefaultSkin` and `AddSkin`) have the same parameter (an `Action<ISkinBuilder>`), but the latter have an additional parameter (the skin name). Using the `ISkinBuilder` you add all the styles of this skin:

* `AddBaseStyle`: This is the "default" style of the skin, the one that is used if no style name is provided.
* `AddStyle`: Adds a named style to the skin.

Both methods use an `Action<IStyleBuilder>` to add the style entries. Here TVision2 offers some interesting things:

1. Style entries can vary based on the current palette or color mode used by the program. This allows you to have a different set of colors if the application runs on 16 or 256 colors.
2. Background does not need to be a fixed color. You can use a `IBackgroundProvider` wich is basically a brush, so you can create gradients and similar backgrounds.

The `IStyleBuilder` offers 2 methods:

* `Default()`: This method allows you to define the base entries, that will be used in any case.
* `When(predicate)`: This method allows you to define entries that will be applied only if the predicate gets true. The predicate could be either a `Func<ColorMode, bool>` or a `Func<IPalette, bool>`.

Here is a example of a skin definition, with a single style. The `builder` is the `ISkinManagerBuilder` object.

```csharp
builder.AddSkin("Mc", sb =>
{
    sb.AddStyle("tvlist", style =>
    {
        style.Default()
            .DesiredStandard(o =>
                o.UseForeground(TvColor.White)
                    .UseBackground(TvColor.Blue)
            )
            .DesiredFocused(o =>
                o.UseForeground(TvColor.White)
                    .UseBackground(TvColor.Blue)
                    .AddModifier(CharacterAttributeModifiers.BackgroundBold)
            )
            .DesiredAlternate(o =>
                o.UseForeground(TvColor.Black)
                    .UseBackground(TvColor.Cyan)
            )
            .DesiredAlternateFocused(o =>
                o.UseForeground(TvColor.Black)
                    .UseBackground(TvColor.White)
        );

        style
            .When(p =>  p.ColorMode ==ColorMode.Direct)
            .DesiredStandard(o =>
                o.UseBackground(() => new VerticalGradientBackgroundProvider(
                    TvColor.FromRGB(0, 200, 100), TvColor.FromRGB(50, 50, 50)))
            );

        style
            .When(p => p.ColorMode == ColorMode.Palettized)
            .DesiredStandard(o =>
                o.UseBackground(() => new VerticalGradientBackgroundProvider(
                    TvColor.FromRGB(0, 255, 0), TvColor.FromRGB(0, 200, 0)))
                );

    });
});
```

We define the `Mc` skin with a single style, called `tvlist`. This style has defined the 4 standard entries (`Standard`, `Active`, `Alternate` and `AlternateActive`) with its foreground and background definitions.

Also, we define two conditional entries, one to be applied if the color mode is `Direct` (fullcolor) and the other to be applied if the color mode is `Palettized` (256 colors). In these cases instead of a fixed background color for the `Standard` entry we use a `VerticalGradientBackgroundProvider` to create a vertical gradient. So, if the color mode is `Direct` or `Palettized`, the `Standard` entry defined previously (using `Default()`) will be overwritten by the `Standard` entry defined in each case.

> Note that the entries are not mixed, they are replaced. So, in this case if the color mode is `Direct`, the background will be the gradient and the foreground will be `TvColor.Black` (the default value), because no `UseForeground` is called in the `DesiredStandard` method chained to `When` (the value of `TvColor.Blue` defined in the `DesiredStandard` method chained to `Default()` is not mixed).
