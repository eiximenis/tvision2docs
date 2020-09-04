---
title: "Keyboard Input"
linkTitle: "Keyboard input"
weight: 7
description: >
  Behaviors: How keyboard works in TVision2
---

Keyboard input has some "limitations" due mainly on how Linux handle the keyboards on terminal:

* No _key up_ event. Only the _key down_ event is available. This is because in Linux terminal there is no keyboard concept: only characters coming to your application.
* Some different key combinations generate the same output character, making impossible for TVision2 to detect keystrokes like `Ctrl+I` (because generates the `TAB` character) or `Ctrl+M` because generates the `CR` generated also by the `ENTER` key.

> Currently if running under Windows, these combinations (like `Ctrl+I`) are properly handled. This can lead to a different behavior of the application when running under Linux or under Windows, and it will be addressed on a future release.

## Processing Keyboard events

You need a behavior to process a keyboard event. In the `Update` method the `BehaviorContext<T>` parameter received contains a `Events` property (of type `ITvConsoleEvents`) which all events (pending to process) for this iteration. You can use the methods of this property to get a keyboard event and inspect it:

```csharp
public bool Update(BehaviorContext<TControlState> updateContext)
{
    if (!updateContext.Events.HasKeyboardEvents) return false;
    var keyEvents = updateContext.Events.KeyboardEvents;
    var consoleKeyInfo = evt.AsConsoleKeyInfo();      // Returns the event as a System.ConsoleKeyInfo struct
    if (...) {
      evt.Handle();       // Mark this event as "Handled". 
    }
}
```

## Handling events

Events can be marked as "handled" by calling its `Handle()` method. When a keyboard event is marked as "handled" it is no longer returned in the `ITvConsoleEvents.KeyboardEvents` property, so it won't be available for the next behaviors. A behavior can process an event and not call its `Handle()` method. In this case, this same event will be available to the next behavior, so it could be processed twice (or more).

As getting the first available (non handled) keyboard event is a typical scenario the `ITvConsoleEvents` interface provides a convenient method called `AcquireFirstKeyboard`:

```csharp
var @event = events.AcquireFirstKeyboard(evt => evt.AsConsoleKeyInfo().Key == _hotkey, autoHandle: true);
```

You need to pass a predicate, and the method will return the first non handled event that satisfies this predicate (if any). The `autoHandle` parameter controls if the event is marked as handled when is returned. If its value is `false` you need to manually handle the event (calling `Handle()`) if appropiate.