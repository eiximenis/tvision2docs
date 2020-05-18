---
title: "Creating child controls"
linkTitle: "Creating child controls"
weight: 2
description: >
  Creating and dealing with child controls
---

When a control creates another control (for performing some task), the other control is usually a "child control" (although TvControls does not enforce that). Usually the `AddAsChild` method is used to add the new control.

You can add a child control whenever you need, but there are some important things to keep in mind. Let's see them all :)

## Creating the child control

To add a child control you need a reference to the _components tree_. You can obtain this reference through the `OnControlMounted` method:

```csharp
private IComponentTree _componentTree;
protected override void OnControlMounted(ITuiEngine engine)
{
    _componentTree = engine.UI;
}
```

When you need you can use the `AddAsChild` method. The following control adds the `menu` control:

```csharp
 _componentTree.AddAsChild(menu, this);
```

>**Important!** Remember that adding a component (thus adding a control too) is an asynchronous operation!

> You can **add a child control before adding the parent control**, but in this case adding the child control will be deferred until the parent is added.

## Setting the focus to a newly created component

There are several ways to do it. But, as adding a control is an asynchronous operation, the **most obvious way to create a control and set the focus to it, do NOT work**:

```csharp
 _componentTree.AddAsChild(menu, this);
 menu.Metadata.Focus(); // probably won't work because menu could not be added yet!
```

You need to ensure that the component is created before setting the focus. A simple way is to add the `OnComponentMounted` event to the child control to focus the control when added:

```csharp
_list = new TvList<MenuEntry>(builder);
_list.AsComponent()
    .Metadata.OnComponentMounted
    .AddOnce(ctx =>
    {
        _list.Metadata.Focus();
        // Can use also: ctx.ControlMetadata().Focus() if not reference to control is available
    });
```

## Capturing the focus

A control can capture the focus. When the focus is captured **only itself and their descendants can be focused**. A typical example of capturing the focus is creating a Dialog. When the dialog is shown, you want to iterate only over the controls of the dialog. 

A control can capture the focus by calling `CaptureFocus()` from its `Metadata` property.

> `CaptureFocus` **do not** perform any focus change. If the focus is set to a control that is not descendant of the control that captured the focus, this control will be focused after calling `CaptureFocus`. However when the focus changes for any reason (like user pressing `TAB`) only the control that called `CaptureFocus` and its descendants could be focused.