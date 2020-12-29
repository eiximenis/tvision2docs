---
title: "Skins"
linkTitle: "Creating Skins"
weight: 2
description: >
  Creating and defining application skins
---

To create an Skin, you must do it when calling the `AddStyles` method:

```csharp
builder.UseTvision2(setup =>
{
    setup.AddStyles(
            sk => sk.AddDefaultSkin(smb =>
            {
                smb.AddStyle("tvgrid", sb =>
                {
                    sb.Default().DesiredStandard(s => 
                        s.UseForeground(TvColor.Green)
                            .UseBackground(() => new VerticalGradientBackgroundProvider(TvColor.FromRGB(0, 255, 255), TvColor.FromRGB(128, 20, 20))));
                });
            })
        )
```

The parameter `sk` is an instance of `ISkinManagerBuilder` which allows you to define an skin in a very expressive way:

* `AddDefaultSkin`: Adds the "default skin" of the application.
* `AddSkin`: Adds an skin to the application

> Each skin has a name that you must specify when creating the skin. `AddDefaultSkin` simply adds an Skin named `Default`.

Both methods (`AddDefaultSkin` and `AddSkin`) take an `Action<ISkinBuilder>` as a parameter, to allow you define the skin. `ISkinBuilder` offers following methods:

* `AddBaseStyle`: Adds the base style to the skin. This style contains base properties, that will be used if specified style is not found in the skin.
* `AddStyle`: Adds the specified style. You must pass the name of the style.

Both methods take an `Action<IStyleBuilder>` as a parameter. The interface `IStyleBuilder` offers methods to define the style.

## Defining styles

If you plan to support various color modes, probably want to use different styles based on the color mode. One of the goals of Tvision2.Styles is abstracting the rest of the application of the color mode used. That means that you can define that specified style can have one foreground if color mode is ANSI 4 bit, or use another color if color mode is 8 bits. Then, based on the color mode of the application Tvision2.Styles will select the appropiate entry automatically.

To support this definitions based on the color mode, `IStyleBuilder` offers the (overloaded) method `When`. This method can take either a `Func<ColorMode, bool>` or `Func<IPalette, bool>` which acts a predicate to define specific style entries to be used if the color mode or the palette of the application satisfy the predicate. To define entries that are applied always you have to use the `Default` method:

```csharp
sb.AddStyle("mystyle", style =>
{
    style.Default().DesiredStandard(o =>
        o.UseForeground(TvColor.White).UseBackground(TvColor.Blue)
    );
    style
        .When(p =>  p.ColorMode == ColorMode.Direct)
        .DesiredStandard(o =>
            o.UseForeground(TvColor.FromRGB(0, 200, 100)).UseBackground(TvColor.FromRGB(0,0,50))
    );
});
```

This snippet defines an style named `mystyle` which:

* Will have foreground (0,200,100) and background (0,0,50) if `ColorMode.Direct` is enabled (application runs in full color).
* Will have white foreground over blue background if previous condition is not met (application runs either in basic color mode or palettized).

After calling `Default` or `When` you **must call** the method `DesiredStandard` to define the standard style value. After calling `DesiredStandard` you can call also:

* `DesiredFocused`: To define the active entry value
* `DesiredAlternate`: To define the alternate entry value
* `DesiredAlternateFocused`: To define the alternate active entry valu
* `Desired`: To define an entry with specific name

> Calling these methods is optional. If not called the value of this entries will be the same as the standard entry.