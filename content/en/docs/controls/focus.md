---
title: "Focus management"
linkTitle: "Focus management"
weight: 4
description: >
  Handling the focus
---

The controls have a new concept that the standard components are unaware of: the focus. In TVision2 Core there is no concept of "focused" components. **All components receive all pending events**. So, TvControls has to implement the focus management itself in a independent way, and it does basically through the `ControlsTree`.

## The controls tree

The controls tree is just an structure in memory that contains all controls and their relationships. It is an analoug of the _Component tree_ but only for controls.

The controls tree is synchronized with the components tree: when a component is added/removed from the components tree, it is automatically added/removed from the controls tree, if this component is wrapped by a control. The controls tree is needed because TvControls needs a efficient way to mantain a list of controls only.

You **never add or remove a control from the controls tree** (there are no methods for doing that). You always add the controls to the components tree (as any other component) and then let the controls tree to synchronize automatically.

> The controls tree use the components tree events (like `OnComponentRemoved` and `OnComponentsTreeUpdated`) to perform this synchronization.

## Handling the focus change key

All UI based applications have some way to change the current focused control (usually this is done by pressing the `TAB` key). When you configure TvControls, a _Hook_ is installed for you. This _Hook_ intercepts all `TAB` keystrokes and interacts with the controls tree to perform a focus change.

> Remember that a _hook_ is, basically, an event handler that is guaranteed to run **before any component behavior**.

## What is a focused control?

The concept of "focused control" is more complex that it seems. Let's summarize concepts:

1. **Only one control** has the focus at each time
2. There could be controls that **never** can have the focus
3. **This is the only control** that receive the events, however...
4. ... events bubble up to the parent control if not handled
5. Focused control **usually displays it different** showing some highlight colors or indications that it has the focus, however...
6. ... sometimes we want a control that is **not really focused** render itself as focused. In thoses cases we say the control is "virtually focused".

> A "virtually focused" control is a control that do not have the focus itself but is rendered as it did.

A good example of last point is  a menu bar. Menu bar can be the really focused control, but when we open a menu from the menu bar, the control goes to the menu itself. But we want the menu bar still renders as focused.

Really using `TvMenuBar` the situation is a even a little more complex: The menu bar creates a `TvMenu` control to display the menu and transfer the focus to the `TvMenu`. However the `TvMenu` is a non-focusable control, that creates a child `TvList` control to show the menu. At this point the status is:

* `TvList` is the real focused control (`ControlsTree.CurrentFocused()` will return that control).
* `TvMenu` do not have focus never (it is created as a non-focusable control)
* `TvMenuBar` still displays as focused, because it is "virtually focused" because one descendant control (the `TvList`) has the real focus.
* When pressing a key, the event goes to the `TvList` control (is the real focused control). If `TvList` control do not process the event, it bubbles to the `TvMenu`. If `TvMenu` do not process the event it bubles to the `TvMenuBar`. To handle the event the associated behavior **must call** the `Handle()` method of the event received, to mark it as "handled". Note that one behavior can perform some action when it receives an event, and if the `Handle()` method is not call, the event will still bubble up. This allows scenarios where both, child and parent must do something on a given event.

## Create virtually focused controls

To create a virtually focused control you must override the `ConfigureMetadataOptions` method of `TvControl` and call the `IsFocused()` method in the `options` parameter. This method gives you access to the focus configuration object (`ITvControlMetadataWhenFocusedOptions` interface), and you must call **only one** of the following methods:

* `Never()`: Control never has the focus. It is non-focusable.
* `OnlyWhenAnyChildHasFocus()`: Control is non-focusable, but is virtually focused if any of its descendants is the focused control.
* `OnlyWhen(Func<bool> hasFocus)`: Control is non-focusable, but is virtually focused when the predicate `hasFocus` evaluates to `true`.
* `WhenItselfOrAnyChildHasFocus()`: Control is focusable and it is virtually focused when itself or  any of its descendants is the focused control.
* `WhenItselfOr(Func<bool> hasFocus)`: Control is focusable and it is virtually focused when itself is the focused control or the predicate `hasFocus` evaluates to `true`.

```csharp
protected override void ConfigureMetadataOptions(TvControlMetadataOptions options)
{
    options.IsFocused().WhenItselfOrAnyChildHasFocus();
}
```

> If you don't call `IsFocused()` the control is focusable it is virtually focused only when it has the real focus.

## FocusTransferred vs IsFocused

The `Metadata` property of a control contains two properties to know if a control is focused:

* `IsFocused`: Returns `true` if the control is the focused one **or** if the control is virtually focused.
* `FocusTransferred`: Returns `true` if the control is the focused one.

> At a given time only one control can have the `FocusTransferred` property to `true`, but more than one control can have the `IsFocused` property to to `true`.

## Handling the focus manually

Sometimes you need to **handle the focus maually**: force some specific control gain focus, or avoid the focus going to specific set of controls. If you are creating complex controls this is usually needed. 

The `Metadata` property of a control offers some methods to manual control the focus:

* `CaptureFocus()`: Capture the focus (only child controls can be focused)
* `ReleaseFocus()`: Release the focus (if was captured by any control)
* `ReturnFocusToPrevious()`: Returns the focus to previously focused control
* `IsFocused`: Returns `true` if control has focus or is virtually focused
* `Focus(bool force = false)`: Transfers focus to the current control. If `force` is `true` focus is transferred regardless of the `CanFocus` property value.
* `CanFocus`: If `true` the control can have the focus. If `false` the control is non-focusable.
* `FocusTransferred`: Values `true` if the control is the one currently focused
* `IsFocused`: Values `true` if the control is the one currently focused or if the control is virtually focused.

## Accessing the controls tree

You usually won't need access to the _controls tree_. However you can access it through the extension method `GetControlsTree()` from `ITuiEngine`. The _controls tree_ exposes the event `FocusChanged` raised when the focus change.