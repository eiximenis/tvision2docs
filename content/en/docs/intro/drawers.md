---
title: "Drawers"
linkTitle: "Drawers"
weight: 5
description: >
  Creating drawers
---

A drawer is responsible to draw a component. Every component that has to be shown needs to have at least one _drawer_ attached to it.

The simplest way to add a drawer is use the `AddDrawer` overload that accepts a delegate:

```
myComponent.AddDrawer(ctx =>
{
    ctx.DrawStringAt(ctx.State.Value, TvPoint.Zero, new TvColorPair(TvColor.Red, TvColor.White));
});
```

This delegate accepts a parameter of `RenderContext<T>` (where `T` is the type of component's state). You can use the methods of the `RenderContext<T>` to effectively draw the component.

## The ITvDrawer<T> interface

The other way to add a drawer to a component is to use the `AddDrawer` method with an instance of an object that implements `ITvDrawer<T>` interface. This interface has a single method:

```csharp
void Draw(RenderContext<T> context);
```

The `Draw` method use the `RenderContect<T>` to effectively draw the component.

## The Render Context

Your code **never draw directly on the console, nor on the viewport**. You have to use always the `RenderContext<T>` object to render your component. The `RenderContext<T>` class offers following methods and properties:

* `State` (of type `T`): State of the component
* `DrawStringAt()`: Allows drawing a string in the desired location with desired colors
* `DrawChars()`: Draws a single character a number of specified times in the desired location with desired colors
* `Clear()`: Clears the viewport
* `Fill()`: Fills the viewport with spaces of specified color

For example, you can use `DrawStringAt()` to draw one string starting in the top-left of the viewport with red letters over a white foreground:

```csharp
ctx.DrawStringAt(ctx.State.Value, TvPoint.Zero, new TvColorPair(TvColor.Red, TvColor.White));
```

Note that the position you use (`TvPoint.Zero` in this example) is always relative to the viewport. **You always draw inside a viewport**, and you can't draw outside it.

## Adaptative drawing

It is possible to add drawers that are only applied if the viewport matches some specific filter. We call this "adaptative drawing". To enable adaptative drawing you must use the `IfViewport` method of the `TvComponent<T>` class, and then chain calls to `AddDrawer`. All drawers added after one `IfViewport` call will be only invoked if the viewport matches the condition specified in the `IfViewport`:

```csharp
helloWorld = new TvComponent<int>(0, "label");
helloWorld.AddDrawer(ctx =>
{
    ctx.DrawStringAt($"Current value: {ctx.State}".PadRight(ctx.Viewport.Bounds.Cols), TvPoint.Zero, new TvColorPair(TvColor.Blue, TvColor.Yellow));
});

helloWorld
    .IfViewport(v => v.Bounds.Cols < 2)
    .AddDrawer(ctx =>
    {
        ctx.DrawStringAt("*", TvPoint.Zero, new TvColorPair(TvColor.Red, TvColor.Green));
    });

helloWorld
    .IfViewport(v => v.Bounds.Cols >= 2 && v.Bounds.Cols <= 5)
    .AddDrawer(ctx =>
    {
        ctx.DrawStringAt(ctx.State.ToString().PadRight(ctx.Viewport.Bounds.Cols), TvPoint.Zero, new TvColorPair(TvColor.Blue, TvColor.Yellow));
    });

```  

In this code we add three drawers to the component. Two of them are adaptative drawers (added after a `IfViewport` call). The other one is a standard drawer.

Then based on the viewport drawn, one of the three drawers will be selected.

> First the adaptative drawers are checked. If the viewport can't be drawn by any of the adaptative drawers then the standard drawers are used.

The conditions to all `IfViewport` do not need to be mutually exclusive. If one viewport matches more than one condition all adaptative drawers selected by all matched conditions will be called to draw the component.


