---
title: "Console drivers"
linkTitle: "Console drivers"
weight: 2
description: >
  What are and how work the console drivers
---

The _console driver_ is the responsible to flush the real characters to the underlying console and to read the events from the terminal. As an application developer you never interact directly with the _console driver_ (it is user under the hoods by TVision2), but you still has to select what _console driver_ to use for your application.

* `DotNetConsoleDriver`: This driver uses standard netcore classes (mostly `System.Console`) to interact with the real terminal. Its strongest point is that it supports all operating systems where netcore is supported. Its weakness is only supports 16 colors and have no mouse support.

* `NcursesConsoleDriver`: This driver uses the [NCurses](https://en.wikipedia.org/wiki/Ncurses) library to adapt to the available terminal. Currently only works in Linux and supports from B/W to 256 colors.

* `Win32StdConsoleDriver`: This driver uses the [Win32 Console API](https://docs.microsoft.com/en-us/windows/console/console-functions) to interact with the console. It supports any version of Windows but only up to 16 colors and has mouse support.

* `Win32AnsiSeqConsoleDriver`: This driver uses the new _virtual terminal_ API that is available in Windows 10. It supports true color and mouse support, but only in Windows 10+.

* `AnsiLinuxConsoleDriver`: This driver uses standard ANSI escape sequences for Linux that should work on most of the modern terminals (like xterm).

## Choosing one console driver

The **best option** is let TVision2 to choose the best console driver at runtime. This will ensure that a valid console driver is chosen for the OS that is running your app. You can use the `UsePlatformConsoleDriver` of `Tvision2Setup`:

```csharp
builder.UseTvision2(setup =>
{
    setup.UseDotNetConsoleDriver();
```

By default this method will defer to `NcursesConsoleDriver` in Linux and `Win32StdConsoleDriver` in Windows.

> **And MacOS?** Currently **MacOS is not officially supported. However is in the roadmap**.

You can change this behavior by passing an options delegate to the `UseDotNetConsoleDriver` to apply some conditional configuration (in run-time):

```csharp
setup.UsePlatformConsoleDriver(opt => 
    opt.Configure()
        // Linux specific config
        .OnLinux(lo =>
        {
            // We want to setup our palette
            lo.UseAnsi().UsePalette(p =>
            {
                // Will init the palette using our terminal name (currently only xterm-256color is supported)
                p.LoadFromTerminalName();
                /* We want to be able to use RGB colors
                but if we are in palette mode (no full direct color) 
                we need to setup a translator that translates any RGB
                color in a palette color. */
                p.TranslateRgbColorsWith(new InterpolationPaletteTranslator());
            });
        })
        // Windows specific config
        .OnWindows(w =>
        {
            // We want to use the new ANSI sequences available on Win10
            w.UseAnsi();
        })
    )
```

