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
DrawResult Draw(RenderContext<T> context);
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

## The DrawResult object

All drawers of a component "share" the same Viewport. Each drawer could be unaware of the other drawers in the same component, thus resulting in two drawers trying to update the same viewport zone. The DrawResult is a way for a drawer to update the viewport that will be passed to the following drawers.

The canonical example for this is a drawer that draws a border on the viewport of a component. When a border is drawn, the first and last row alognside the first and last column of the viewport are occuped with the border. Imagine we have a component with this drawer:

```csharp
helloWorld.AddDrawer(ctx =>
{
    ctx.DrawStringAt("Hello World".PadRight(ctx.Viewport.Bounds.Cols), TvPoint.Zero, new TvColorPair(TvColor.Blue, TvColor.Yellow));
});

```

This drawer renders the string "Hello World" plus needed spaces to fill all the viewport length. The drawer draws the string beginning at location `TvPoint.Zero` which is starting at the first row and column of the viewport.

If this component has another drawer that renders a border, the border would be rendered also in the first row, and the final results would be wrong. In this case we need the "Hello World" string to be drawn at second row, and also starting in the second column (not the first). The usage of `DrawResult` allows handling these situations.

In this case the drawer that renders the border **must run before** (needs to be before in the list of drawers of the component). And this drawer should return a `DrawerResult` like:

```csharp
new DrawResult(TvPoint.FromXY(1, 1), TvBounds.FromRowsAndCols(2, 2));
```

That means that following drawers of the component will have a new viewport that

* Will start at the position (1,1) of the original viewport
* Will have 2 rows minus than the original viewport
* Will have 2 columns minus than the original viewport

So, even our drawer is still using `TvPoint.Zero` when rendering the "Hello world" string, the string will be drawn _inside_ the border.

> If the drawer does not want the viewport of following drawers to be adapted, need to return `DrawResult.Done`.

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

## Reusing drawers between components

Reusing drawers between components is not as straightforward as it could seem because **drawers are tied to the component's state type**. If you have two components named t1 and t2, that are instances of `TvComponent<T1>` and `TvComponent<T2>` you can't directly reuse the drawers even though `T2` were subtype of `T1`.

Drawers for t1 have to implement `ITvDrawer<T1>` and drawers for t2 have to implement `ITvDrawer<T2>` and there is no direct conversion between these two interfaces, despite the relationship between `T1` and `T2`. However if you want to reuse some drawers between components, you can do it, thanks to the method `AddDrawer<U>`. This method allows you to register a drawer that draws a different state type, and TVision2 cares about the type conversion. The signature of this method is:

```csharp
public TvComponent<T> AddDrawer<U>(ITvDrawer<U> drawer, Func<T, U> stateConverter);
```

You need to pass the drawer that you want use (note that we are using a `TvComponent<T>` but this parameter is a `ITvDrawer<U>`) and the converter that TVision2 must use to convert the state from the component type, to the type expected by the drawer.

> Using a drawer that is for other state types, has a little performance hit, because **a new RenderContext<U> must be created**, with the "new state" for this drawer (created by invoking the `stateConverter` funcion).