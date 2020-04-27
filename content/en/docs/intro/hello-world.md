---
title: "Hello world!"
linkTitle: "Hello World"
weight: 1
description: >
  No software documentation is really a software documentation without a 'Hello World' sample
---

Let's see how we can create a "Hello World" program using TVision2. First, the code:

```csharp
using Microsoft.Extensions.Hosting;
using System;
using System.Threading.Tasks;
using Microsoft.Extensions.DependencyInjection;
using Tvision2.Core;
using Tvision2.Core.Colors;
using Tvision2.Core.Components;
using Tvision2.Core.Render;

namespace Tvision2.HelloWorld
{
    internal class Program
    {
        private static async Task Main(string[] args)
        {
            var builder = new HostBuilder();
            builder.UseTvision2(setup =>
            {
                setup.UseDotNetConsoleDriver();
                setup.Options.UseStartup((sp, tui) =>
                {
                    var helloWorld = new TvComponent<string>("Hello world!");
                    helloWorld.AddDrawer(ctx =>
                    {
                        ctx.DrawStringAt(ctx.State, TvPoint.Zero, new TvColorPair(TvColor.Blue, TvColor.Yellow)) ;
                    });
                    helloWorld.AddViewport(new Viewport(TvPoint.FromXY(10, 10), 30));
                    tui.UI.Add(helloWorld);
                    return Task.CompletedTask;
                });
            }).UseConsoleLifetime();
            await builder.RunTvisionConsoleApp();
        }
    }
}
```

This code the `Hello world!` string at position (10,10) with a blue foreground and a yellow background. Yes, probably you are thinking "that's a lot of code for just printing a string" and you are true, but this is just because the sample is too simple to show all TVision2 power. But anyway, let's see the main components of it.

First, you need to configure some aspects of TVision2 before starting your app. This is done through the extension method `UseTvision2` of the `IHostBuilder` interface. That method expects an ` Action<Tvision2Setup>` with the setup code. In this case we are using the `UseDotNetConsoleDriver` that forces TVision2 to use the pure netcore-based console driver. A console driver is how TVision2 will interact with the real console (as you will see, as a developer, you never access directly to the console). There are various console drivers, each one with their weaknesses and strenghtnesses.

Then, we are using the simplest way to start the application which is using the `UseStartup` method to provide the code of our application. On more complex applications another way (using a separated _startup_ class) is preferred. When using `UseStartup` you have access to two parameters: an `IServiceProvider` which gives you access to some TVision2 objects and a `ITuiEngine` wich holds the TVision2 engine. In the `UseStartup` method we do following things:

1. Create a `TvComponent<string>` class. This creates a _component_ that holds an `string` object. Components are the main objects in TVision2: your application is composed of a set of components that interact. All components are instances of the **sealed** `TvComponent<T>` class. TVision2 prefers composition over inheritance, so **you can't inherit from `TvComponent<T>`**.
2. Then we add a drawer to our component. A drawer is a piece of code that draws the component to a _viewport_. All drawers use the methods provide in the `RenderContext` class to draw to the _viewport_. In this sample the viewport is located at coords (10,10) and have a bounds of 30 cols and one row.
3. Then we use the `AddViewport` method to add a _viewport_ to the component. A _viewport_ is a region (position, bounds and layer) in the console window. A component can only draw in its own _viewport_.
4. Finally we add the component to the _component tree_ using the `Add` method of the `UI` property of the engine. The component is now added to the tree and TVision2 will begin drawing it.

Congratulations! You have finished your first TVision2 app! Now, we'll do some changes to it to see some of the TVision2 characteristics :)

## Updating a component

If you put a breakpoint in the `ctx.Drawstring` line you'll see that **the breakpoint is only hit once**. This is because once a component is drawn, it won't be drawn again **until it changes**. This raises the question on how TVision2 knows if a component "has changed" and need to be redrawn. Well, a component needs to be redrawn if **its state has changed**. The "state" of a component is the additional object it holds (in our example is a string, because we are using a `TvComponent<string>` object). For changing a state of a component there is a `SetState` method. However **this won't work**:

```csharp
helloWorld.AddDrawer(ctx =>
{
    ctx.DrawStringAt(ctx.State, TvPoint.Zero, new TvColorPair(TvColor.Blue, TvColor.Yellow));
    helloWorld.SetState ("Hello world! " + new Random().Next(1,1000));
});
```

This is **a very bad practice**. You should never access to the component from inside the `AddDrawer` method (in this case you have access to it because this is a sample code, but in more complex applications, usually the drawer do not have access to the reference that holds the component). Anyway, this does not work because **TVision2 assumes that drawing the component NEVER change its state**. TVision2 has an specific way to change the state of a component, which is **using a behavior**. We will use the `AddStateBehavior` method to add a behavior to a component:

```csharp
helloWorld.AddStateBehavior(s => "Hello world " + new Random().Next(1, 1000) + "    ");
```

If you run the application again you will see how the content of the label is changing continuously.

> Note that we don't call the `SetState` method of the `TvComponent`. The `SetState` method is rarely used, but it is provided to some advanced scenarios. The standard way to update a component is using a behavior.

A component **can have zero or more behaviors** and there are many ways to add a behavior to a component. We have used the `AddStateBehavior(Func<T,T>)` method which is the simplest one, but there are other ways. We will discuss all of them in the behaviors section.

> Remember: To change the state of a component **try to use always a behavior**. Use `SetState` method, only when you need to change the state of another component.

## Adding more than one viewport

A component can have more than one viewport:

```csharp
helloWorld.AddViewport(new Viewport(TvPoint.FromXY(10, 10), 30));
helloWorld.AddViewport(new Viewport(TvPoint.FromXY(10, 11), 30));
helloWorld.AddViewport(new Viewport(TvPoint.FromXY(20, 13), 30));
``` 

If you run the code, you will see **three labels** all with the same content and changing at the same time.
