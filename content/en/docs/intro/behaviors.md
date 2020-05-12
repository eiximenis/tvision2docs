---
title: "Behaviors"
linkTitle: "Behaviors"
weight: 3
description: >
  Behaviors: How to work with them
---

The _behaviors_ contains the **logic** of the component. A single component can have more than one behavior. All behaviors share the state of the component. There are different ways to add a behavior to a component to cover different needs. Let's start with the simplest one...

## AddStateBehavior (new state behavior)

This method is, probably, the simplest way to add one behavior to one component:

```csharp
helloWorld.AddStateBehavior(s => "Hello world " + new Random().Next(1, 1000) + "    ");
```

This method creates a behavior **updates the full state** of the component. At each iteration the delegate is called and the state of the component is **replaced** with the state returned by the delegate. Most times this will force the component to redraw itself.