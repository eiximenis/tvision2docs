---
title: "Using existing controls"
linkTitle: "Using existing controls"
weight: 3
description: >
  How to use existing controls
---

To create a control you have to create an instance of its class. For example you need to create a label you have to create a `TvLabel` object. To create a control you need to specify 3 different values:

* The control options
* Its initial state
* The control configuration

The options and the state depends on the control type, but the control configuration includes stuff that is the same for every control, like the _viewport_ to use and the control name.

All controls have a constructor that accepts either:

* A instance of `TvControlCreationParameters<TState>` or
* A instance of `TvControlCreationParameters<TState, TOptions>`

The 1st is used for controls that do not have options.

> The `TvControlCreationParameters` class contains the options (if control has any), the state and the configuration

## The control options

The options of a control **are immutable** and are usually used in the drawers. They specify specific set of options that are set at the control creation (for example if the text of a label has to be centered). Once the options are set, they could not be changed.

TVision2 does not care about how you end up with a `TOptions` object, however all the controls provided follow these rules:

* `TOptions` is an interface exposing only read-only properties (to ensure immutability)
* For each `TOptions` type there is a `TOptionsImpl` type that implements the interface.
* For each `TOptions` type there is a `TOptionsBuilder` type that provides a nice API that creates a `TOptionsImpl` and return it as a `TOptions` value.
* Every control provides a static method, called `UseParams` that gives access to the `TOptionsBuilder` alongside with additional builders to allow build the state and the configuration.

## The control state

The control state is the _data_ with the control is working with. Each control wrapps a `TvComponent<T>` instance where `T` is the type of the control state.

## The control configuration

The control configuration specifies:

* The name of the control
* The viewport of the control
* The skin to use (if is not the current one)

## Building the options, the state and the configuration

The class `TvControlCreationBuilder` is a multi-builder that allows building the options, the state and the configuration. It is not intended to be used directly, instead each control provides the static method `UseParams` that gives access to a `TvControlCreationBuilder` configurated for the control type:

```csharp
var paramsBuilder = TvLabel.UseParams();
```

Once we have the `paramsBuilder` we can use following methods:

* `Options()`: To configure the control options. This method accepts one `Action<TOptionsBuilder>` as a parameter
* `WithState()`: To specify a initial state for the control
* `Configure()`: To specify the control configuration
* `Build()`: To build the final `TvControlCreationParameters` class

```csharp
var paramsBuilder = TvLabel.UseParams();
var labelParams = paramsBuilder
    .Options(o => o.Center())
    .WithState(LabelState.FromText("Hello World"))
    .Configure(c => c
        .UseControlName("mylabel")
        .UseViewport(new Viewport(TvPoint.FromXY(10, 10), 20)))
    .Build();
var label = new TvLabel(labelParams);
```

This code will create a label showing `Hello World` in the position `(10, 10)` and with the text centered.

### Create the TvControlCreationParameters directly

If you want, you can create the `TvControlCreationParameters` object directly:

```csharp
var options = new TvLabelOptions();
((ITvLabelOptionsBuilder)options).Center();
var labelParams = new TvControlCreationParameters<LabelState, ILabelOptions>(null, 
    new Viewport(TvPoint.FromXY(10, 10), 20), LabelState.FromText("Hello World"), options, "mylabel");
var label = new TvLabel(labelParams);
```

> To use this option you must know the relationship between `TOptions`, `TOptionsImpl` and `TOptionsBuilder` types. In this example you must know that `TOptions` is `ILabelOptions`, `TOptionsImpl` is `TvLabelOptions` and `TOptionsBuilder` is `ITvLabelOptionsBuilder`.

Or a third option is to use the `TvControlCreationParametersBuilder` which is a intermediate builder that attach a configuration to a existing state and options and creates the final `TvControlCreationParameters`:

```csharp
var options = new TvLabelOptions();
((ITvLabelOptionsBuilder)options).Center();
var labelparamsBuilder = new TvControlCreationParametersBuilder<LabelState, ILabelOptions>(LabelState.FromText("Hello World"), options);
labelparamsBuilder
    .UseViewport(new Viewport(TvPoint.FromXY(10, 10), 20))
    .UseControlName("mylabel");
var labelParams = labelparamsBuilder.Build();
var label = new TvLabel(labelParams);
```

All three versions of the code are equivalent, **but the recommended way is to use the first option**.
