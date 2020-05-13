---
title: "Behaviors"
linkTitle: "Behaviors"
weight: 6
description: >
  Behaviors: How to work with them
---

The _behaviors_ contains the **logic** of the component. A single component can have more than one behavior. All behaviors share the state of the component. There are different ways to add a behavior to a component to cover different needs. Let's start with the simplest one...

## AddStateBehavior (new state behavior)

The `AddStateBehavior` method is, probably, the simplest way to add one behavior to one component. The parameter is a delegate `Func<T,T>` where `T` is the type of the component's state. The delegate receives the current state and have to return **a new state**.

```csharp
helloWorld.AddStateBehavior(s => "Hello world " + new Random().Next(1, 1000) + "    ");
```

This method creates a behavior that **updates the full state** of the component. At each iteration the delegate is called and the state of the component is **replaced** with the state returned by the delegate. Most times this will force the component to redraw itself.

## AddBehavior with single delegate

There are various overloads of the `AddBehavior` method. The simplest one is the one that uses a single delegate `Func<T, bool>` as a parameter (again `T` is the type of the component's state). The delegate receives the current state of the component and can do whatever it needs with it. The delegate must return `true` if the component needs to redraw (usually, if the state gets updated). Note that this method do not work with immutable states, because do not replace the whole component state (only updates it).

## ITvBehavior<T> interface

Another option to add a behavior to a component is create a class that implements the `ITvBehavior<T>` (`T` is the type of the component's state). This interface has a single method:

```csharp
bool Update(BehaviorContext<T> updateContext);
```

The method `Update` is where the behavior do its logic. The parameter `updateContext` give access to the current component's state and contains some other useful data. The method must return `true` if the component needs to be redrawn. The `BehaviorContext<T>` class do not contains any method to allow replacing the whole state of the component, so this approach do not work with immutable states.

Once you have one instance of any class implementing `ITvBehavior<T>` you can use the following overload of `AddBehavior`:

```csharp
public void AddBehavior<TB>(TB behavior, Action<BehaviorMetadata<T, TB>> metadataAction = null) where TB: ITvBehavior<T>
```

This method is the preferred way to add more complex behaviors as it gives access to the full `BehaviorContext<T>` object and not only the state.

## The BehaviorMetadata

As you can see the second parameter of the previous `AddBehavior` is an `Action<BehaviorMetadata<T, TB>>` that allows to (optionally) configure the metadata of this behavior. The metadata is used by TVision2 and allows setting behavior options in a independent way. The class `BehaviorMetadata<T,TB>` provides following methods:

* `UseScheduler(BehaviorSchedule schedule)`: This methods sets when the behavior is invoked. The default value is `BehaviorSchedule.OncePerFrame` which means the behavior will be invoked at each cycle. The other option is `BehaviorSchedule.OnEvents` which means that the behavior will only be invoked if there is any event pending to be processed. If your behavior relies on some events (like keystrokes) use `OnEvents` to maximize the performance.
* `AddDependency<TR>(Expression<Func<TB, TR>> propertyExp, string name)`: This method allows adding a dependency property to **another component**. Using dependencies, a behavior object can obtanin references to other components (existing or future) automatically.

## The Behavior Context

When using a custom behavior object that implements the `ITvBehavior<T>` interface, the `Update` method receives a parameter of type `BehaviorContext<T>`. This object contains following readonly properties:

* `State`: Current state of the component (type `T`)
* `ITvConsoleEvents Events`: A collection of all pending events. Can be empty if there are no events left to process (type `ITvConsoleEvents`)
* `ComponentLocator`: A helper object that finds other components. The locator can find the parent component of the current component, the root component or any other component by name (type `ComponentLocator`).

> **Note**: Avoid the use `ComponentLocator.GetByName()` method, as this performs a search by name every time, which can incur in performance hit. If your behavior has a dependency on other component, better use a `behavior dependency`. The method `GetByName` should be used only in cases when you don't want to hold a reference to another component for any reason.

## Behavior dependencies

A behavior can have a dependency to another `TvComponent`. These dependencies are properties of the class that implements the `ITvBehavior<T>` and are set automatically by TVision2. There are two ways to set a component dependency:

* By using the `TvComponentDependencyAttribute` on the property of the behavior class
* By calling the `AddDependency` method of the `BehaviorMetadata`.

Both methods are equivalent, so use what fits better to you.

```csharp
class DemoBehavior : ITvBehavior<FooState>
{     
    [TvComponentDependency(Name ="label")]
    public TvComponent<string> Label { get; set; }
    public bool Update(BehaviorContext<FooState> updateContext)  {...}
}
```

**or** do not use `[TvComponentDependency]` and use `AddDependency`:

```csharp
myCmp.AddBehavior(new DemoBehavior(), m => m.AddDependency(u => u.Label, "label"));
```  

## Behavior factories

In most cases you create the behavior object and use `AddBehavior` to attach the object to the component. However sometimes your behavior can depend on some objects that are in the dependency injection system provided by net core. In these cases you can leverage the creation of the behavior to TVision2 itself. TVision2 will create a behavior by using a `IServiceProvider` allowing to inject dependencies in the constructor of the behavior class.

To create a behavior through a factory you have to use following `AddBehavior` overload:

```csharp
AddBehavior<TB>(Action<FactoryBehaviorMetadata<TB, T>> metadataAction = null) where TB : ITvBehavior<T>
```

The `FactoryBehaviorMetadata<TB,T>` exposes the `UseScheduler` and `AddDependency` methods seen before and following new methods:

* `OnCreate(Action<TB> afterCreate)`: This method allows you to pass a delegate that will be called against the new created behavior. This allows initializing the behavior in any needed way once it is created.
* `CreateUsing(Func<IServiceProvider, TB> creator)`: This methods set the delegate to use for create the behavior. The delegate receives a `IServiceProvider` that you can use to inject the required dependencies in the behavior constructor.

Note that calling `CreateUsing` is optional. If you don't provide a way to create the behavior a standard mechanism is used. This standard mechanism has the following limitations:

1. Behavior class can have only one single constructor
2. All parameters of the constructor must be services registered in the `IServiceProvider`




