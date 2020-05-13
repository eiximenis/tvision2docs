---
title: "Components"
linkTitle: "Components"
weight: 4
description: >
  Working with components
---

Let's see how we can create a simple component for using in our application. To create a component, you must create a `TvComponent<T>` class object. Every component handles a single piece of data (its state) of type `T`.

```csharp
var label = new TvComponent<string>("Hello world", "mylabel");
```

This code creates a component that holds a simple `string` value as state. The component is initialized to have state of `Hello World` and a name of `myLabel`.

Once you have a component you can add to your application using the `UI` property of the `ITuiEngine`:

```csharp
engine.UI.Add(label);
```

At this point the component is attached and running! However you won't see anything at screen. To see something in the screen we need to specify **where** and **how** the component has to be drawn. For the _where_ we use a _viewport_ and for the _how_, we need a _drawer_.

## Viewports

A viewport is a instance of a class that implements the `IViewport` interface and it is, basically, a position and a bounds. The most common way to create viewports is to use the `Viewport` class:

```csharp
var viewport = new Viewport(TvPoint.FromXY(10, 10), 30);
```

This creates a viewport that starts at position `(10, 10)` and has 30 columns and a single row. For multiline viewports there is another constructor that accepts a `TvBounds` object. This constructor also accepts a `Layer` object with the layer of the viewport (its z-index):

```csharp
var viewport = new Viewport(TvPoint.FromXY(10, 10), TvBounds.FromRowsAndCols(10,4), Layer.Standard);
```

Once you have a viewport you can add it to the component using the `AddViewport` method. This method returns the viewport identifier (a `Guid`). The first viewport has always the `Guid.Zero` value and it is considered the _default_ viewport. You need this `Guid` to update a specific viewport.

Note that a component can have more than one viewport. In this case it will be rendered in *all* of its viewports.

## Drawers

A drawer is responsible to draw the component, or a part of it. A component can have zero or more drawers attached to it, and all of them are executed to compose the final output of the component.

The method `AddDrawer` allows adding a drawer to the component.
