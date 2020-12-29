---
title: "TVision2.Styles concepts"
linkTitle: "Concepts"
weight: 1
description: >
  Concepts of Tvision2.Styles
---

Tvision2.Styles has the following concepts:

* Skin (`Tvision2.Styles.ISkin`): A set of styles grouped together. Your application can use one or more skins.
* Style (`Tvision2.Styles.IStyle`): A set of style entries. Each style can have four standard entries named Standard, Active, Alternate and AlternateActive, but also additional entries can be defined.
* Style Entry (`Tvision2.Styles.StyleEntry`): A foreground color, a background color and the character modifiers to apply. The colors can be static (instances of `Tvision2.Core.Colors.TvColor`) or color providers (`Tvision2.Styles.IColorProvider`) which can apply different colors based on the position of each character.
* Skin Manager (`Tvision2.Styles.ISkinManager`): A service offered to access the skins (and styles) defined in the application.

## Adding Styles to your application

To allow the use of Tvision2.Styles you have to enable it when configuring the application (in `Program.cs`) using the extension method `AddStyles` defined in the namesapce `Tvision2.DependencyInjection;`.

```csharp
 builder.UseTvision2(setup =>
    {
        // Previous Tvision2 Configuration
        setup.AddStyles(...);          // Enable styles
    });
```

The method `AddStyles` accepts an `Action<ISkinManagerBuilder>` as a parameter to allow you to define the skins and styles used by the application.

> You can call `AddStyles` several times to define different skins. However a single skin definition cannot be spanned in different calls.



