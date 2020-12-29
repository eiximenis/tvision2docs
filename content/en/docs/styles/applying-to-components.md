---
title: "Skins and components"
linkTitle: "Applying skins"
weight: 2
description: >
  Applying skins to components
---

Applying an style to a component means "select the appropiate foreground and background values" automatically. Of course the drawer is the responsible to use this values accordingly ;-)

To support the use of styles, an extension method over `RenderContex`  named `Styled` is defined. This method takes the name of style and returns an instance of `StyledRenderContext` which drawer can use. Using this new render context will use the style colors when drawing to the console:

```csharp
mycomponent.AddDrawer(ctx =>
{
    ctx.Styled("text").DrawStringAt(ctx.State.PadRight(ctx.Viewport.Bounds.Cols), TvPoint.Zero);
});
```

The string `ctx.State` will be drawn in the console using the foreground and background defined in the `text` style. By default the "Standard" style entry is used.

## Accessing the ISkinManager from a drawer

Currently using `Styled` only gives you access to the "Standard" entry of specific style. For most components this can be enough. However other components must need access to the rest of entries, in this case they need to use the `ISkinManager` to select the Skin and Style directly. To access the `ISkinManager` you can use the extension method `GetSkinManager` (defined over `RenderContext`):

```csharp
mycomponent.AddDrawer(ctx =>
{
    var myColors = ctx.GetSkinManager().CurrentSkin["text"].Alternate;      // myColors is a StyleEntry object with the alternate values of "text" style
});
```



