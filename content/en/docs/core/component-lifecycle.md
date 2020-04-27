---
title: "The component lifecycles"
linkTitle: "The component lifecycle"
weight: 2
description: >
  What is the lifecycle of a component
---

The lifecycle of a component it is very simple. A component can be:

* Dettached: The component object is created but is not attached to the tree. TVision2 don't care about a component until it is attached to the tree.

* Attached & Running: The component is running and it is attached to the tree. When a component is attached participates from the main loop through the methods `Update` and `Draw`, until you remove the component from the tree.

## Events and notifications when attaching a component

When a component is **attached to the tree** following events are triggered:

1. The `OnComponentMounted` action from the component metadata is called.
2. The `ComponentAdded` event from the ComponentTree is triggered
3. The `AfterAddAction` specified when creating the component (if any) is called
4. The `TreeUpdated` event from the ComponentTree is triggered

> Remember that adding a component to the tree is an asynchronous operation and many components can be added in a single "add operation". Steps 1,2 and 3 happen once per component and step 4 happen only once per add operation (when all components are added).

When **dettaching** a component following happens:

1. The `OnComponentWillBeUnmounted` action from the component metadata is invoked. At this point the component can abort the dettach operation.
2. The `OnComponentUnmounted` action from the component metadata is invoked
3. The `ComponentRemoved` event from the tree is triggered
4. The `TreeUpdated` event from the tree is triggered

> Like adding components, removing is an asynchronous operation. Steps 1, 2 & 3 happen once per each component, and step 4 only once when all components in the "remove operation" are removed from the tree.

## Dettaching a parent component

When a component that have childs is dettached, all their childs are dettached too. First the childs are dettached and lastly the parent component is.

> If **any** of the childs aborts the dettaching operation (through the `OnComponentWillBeUnmounted` action) no component is deleted.

## Using the Metadata actions of the component lifecycle

When creating the component you can add the actions `OnComponentMounted`, `OnComponentUnmounted` and `OnComponentWillBeUnmounted`:

```csharp
var cmp = new TvComponent<string>("hello", "label", 
    cfg =>
    {
        cfg.WhenComponentMounted(ctx => ComponentMounted(ctx));
    });
);
```

This code creates a component holding an string, and adds the `ComponentMounted` action. When adding an action you provide a deletage that will be invoked.

You can use following methods:

```csharp
WhenComponentMounted(Action<ComponentMoutingContext> mountAction);
WhenComponentUnmounted(Action<ComponentMoutingContext> unmountAction);
WhenComponentWillbeUnmounted(Action<ComponentMountingCancellableContext> unmountAction);
WhenChildMounted(Action<ChildComponentMoutingContext> childMountAction);
WhenChildUnmounted(Action<ChildComponentMoutingContext> childUndmountAction);
```

* `WhenComponentMounted` (`OnComponentMounted` action): Invoked when the component is mounted
* `WhenComponentUnmounted` (`OnComponentUnmounted` action): Invoked then the component is unmounted
* `WhenComponentWillbeUnmounted` (`OnComponentWillbeUnmounted` action): Invoked when the component is about to be invoked giving the chance to the component to abort the operation
* `WhenChildMounted` (`OnChildMountedAction` action): Invoked when a child component is mounted 
* `WhenChildUnmounted` (`OnChildMountedUnmounted` action): Invoked when a child component is unmounted.

> When adding a component as a child, it is possible to use an option to avoid the `OnChildMountedAction` to be called.
