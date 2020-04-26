---
title: "The Component Tree"
linkTitle: "The component tree"
weight: 1
description: >
  What is the Component Tree and how to work with it.
---

The _component tree_ is a structure in memory that holds all the _components_ and their relationships. There is a single _component tree_ per each TVision2 application, and you can access to it using the `UI` property from the `ITuiEngine`. The _component tree_ is responsible for:

* Holding all _components_ of the application
* Mantaining their relationships

## Adding components to the tree

You can create a _component_ (instance of `TvComponent<TState>` class) at any time, but until you don't add the component to the _component tree_ it do not exists as for TVision2 concerns. Howerver creating a component and adding it to the tree are two separate steps.

When you add a component to the tree **you specify the relationship to this component from other component**. This is a very important concept in TVision2, so take time to assimilate it: the relationships between components are defined when you add components to the tree. You can create parent/child relationships but the components participating in those relationships are unaware of that: the relationship is maintained in the tree, not in the _component_ itself.

You can add a component in the tree by calling one of the _Add_ methods:

```csharp
public TvComponentMetadata Add(TvComponent componentToAdd, Action<AddComponentOptions> addOptions = null);
public TvComponentMetadata AddAfter(TvComponent componentToAdd, TvComponent componentBefore);
public TvComponentMetadata AddAsChild(TvComponent componentToAdd, TvComponent parent, Action<IAddChildComponentOptions> options = null);
```

The `AddAfter` and `AddAsChild` methods are just offered for convenience as you could do everything using only the `Add` method. 

> You **can** add a child component before adding its parent, but the operation won't be effective until you add the parent.

By default if a component is added as a child of another component:

1. If the parent component is not added yet, the add operation is delayed until the parent component is added.
2. The parent component is notified when the child is added
3. When the parent component is deleted, all their childs are deleted too
4. A child component _can_ prevent the deletion of its parent. In this case no components are deleted.

When adding a component using the  `Add` method, you can use the `addOptions` parameter to specify additional options like if this component has parent (same as calling `AddAsChild`), if it has to be added after other component (same like `AddAfter`) or pass some aditional action to be performed when the component is effectively added.

```csharp
componentTree.Add(mycomponent, options =>
{
    options.WithParent(parentComponent, childOpt =>
    {
        childOpt.DoNotNotifyParentOnAdd();
        childOpt.RetrieveParentStatusChanges();
    });
});
```

This code adds the `mycomponent` as child of `parentComponent` and sets the additional options to not notify the parent component that this component is added as its child and that they need to be invalidated if its parent component gets invalidated too (this is useful if the state of the child component have some dependency to the state of its parent).

> Remember: Adding components is an **asyncronous operation**. Once the `Add` method finishes the component is not yet added in the _component tree_ (just queued to be added).

## Removing components from the tree

Removing a component from the tree is performed by calling the `Remove` method:

```csharp
bool Remove(TvComponent component);
```

Like adding a component, removing a component is an asynchronous operation. The `Remove` method just queues a component to be removed. The method returns `false` if the component is not found.

> When a component is deleted all their childs are deleted too. **First are deleted the childs and finally the component itself**.