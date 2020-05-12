---
title: "Colors"
linkTitle: "Colors"
weight: 3
description: >
  Working with colors
---

Dealing with colors is complex, because there are important differences between *NIX and Windows. 

## Color Modes

Your application can run in one of this four colour modes:

* `NoColor`: No color support. Only B/W
* `Basic`: Basic color support (8/16 colors)
* `Palettized`: Palettized support. TVision2 supports palettes up to 256 colors.
* `Direct`: Direct RGB colors (aka true color)

The API for colors is built upon the `TvColor` type. The `TvColor` abstracts three APIs in one:

1. The RGB API, used when working in direct (full color) mode
2. The palette API, used when working in palettized modes (like 256 colors)
3. The index API, used when working in fixed-color modes (like 16 colors)

The **final color mode depends on the operating system, the terminal and your configuration**:

If application is running in Windows 7 your application will always be in Basic mode. There is no way, currently, to override this configuration (althought it is in the roadmap, because this would allow run Windows applications in 256 modes through ConEmu or similar). If application is running in Windows 10 you can choose between run Basic, Palettized or Direct color modes.

In Linux, if the application uses the NCurses console driver (to support old terminals) the color mode can be NoColor, Basic or Palettized. No Direct support is provided through NCurses. To support Direct color mode, you have to use the Ansi console driver (and in this case your application can run in Direct, Palettized or Basic based on your configuration).

## Enabling True Color

If your OS is capable to use true color (Linux or Windows 10) you can enable true color in your application. This allows the use RGB colors (created using `TvColor.FromRGB`). An RGB color is created by using 3 bytes (one for red, one for green, one for blue).

> To enable true color in Linux you must use the Ansi console driver, there is **no support for true color when using NCurses**:

Use following code to enable true color in your application:

```csharp
setup.UsePlatformConsoleDriver(opt => opt
    .Configure()
    .OnLinux(l => l.UseAnsi().EnableTrueColor(tc => tc.WithBuiltInSequences())
    )
    .OnWindows(w => w.UseAnsi().EnableTrueColor())
  ); 
```

> In Linux your application can run under a terminal with NO true color support. If you enable true color and use RGB colors when running in a terminal with no true color support, your application will output garbage. Currently TVision2 do not perform any check to ensure that terminal supports true color (and note that there is no any standarized way to do it). We will implement some checks in the future (like reading `COLORTERM` environment variable).

If you are in RGB color mode, can create a `TvColor` by calling `TvColor.FromRGB` method. If you are in palettized mode, you can use `TvColor.FromPaletteIndex` and if you are in fixed-color mode you can use `TvColor.FromRaw`. Let's discuss all these methods a little further.

## Using palettized color mode (up to 256 colors)

Most terminals support a 256 colors palette (in Linux this is probably the most common color mode. Most known terminals like `xterm-256` work this way). In this mode a color is just an index (from 0 to 255). Usually the first 16 colors are the same that the basic color mode and the colors range from 232-255 are different levels of grey. However every terminal can have its own palette and most terminal emulators let the user define its own palette. To create a palette color use the `TvColor.FromPaletteIndex` method.

TVision2 offers several capabilities when working with palettes. TVision2 creates a palette object (`IPalette` instance) to help you dealing with the palettes. A palette contains **a set of colors, with an index, a name and also its RGB components**. At startup your application can choose how to fill the palette:

1. Reading the current palette of the terminal
2. Reading a definition file

If you read a palette from a definition file, the palette of your application won't be (probably) the same palette used by the terminal. Note that **always the palette of the terminal is the one that is used** (that makes sense, as is the terminal the one who renders the application). If your application has one palette and the terminal has another palette, your application can draw something using the color 120 thinking that is a "pink", and the final color rendered in screen will be the color 120 of the terminal palette (which can be any other color). However **you can change the palette of the terminal to fit the palette of your application**. Note that, in Linux, not all terminals support redefining its palette.

A palette has another possibility: as it contains the RGB components of each color, it is possible **to translate an RGB color to a palettized color**, that is, finding the color in the palette that is more similar to the RGB color passed. This allows to use RGB colors even though in terminals that do not support true color, but the final colors can be slightly different (as the color used is always one palette color). TVision2 allows you to configure how to convert from RGB colors to palette colors.

> Technically it is possible to use the palette to transform from a palette color to a RGB one, but this is not useful, because all terminals that support true color, support palette too. So, **you can use a palette color in a true color application**. If a palettized color is used when the application is configured in true color mode, it will work as expected.

### Using ANSI colors in palettized mode

 In palettized mode, a color is simply an integer with the index of the color in the palette. So, **based on your palette** the color index 120 can be red, lime or turquoise. Fortunately **most palettes have the same first 16 values like the Ansi 4 bits colors**. If this is the case, you can use a `TvColor` like `TvColor.Red` in a palettized application and the result color will be Red. If the first colors of the palette are not like the ANSI standard, you can sill use `TvColor.Red` but there is no guarantee that the real color will be red!

In a palettized color mode, the palette can be "fixed" or "configurable". In a "configurable" palette you can choose the colors you want to use from a boarder range of colors (like choosing 256 colors from a range of millions). In a "fixed" palette, the palette is, well, fixed by the terminal, and can't be changed. When working in a palettized mode the main question is "how I can know what color is the color index 120?".

### The Palette object

TVision2 offers the `IPalette` interface for this question. And **to simplify your application development, TVision2 will ALWAYS offer you a palette, even though your application is running in a fixed color mode or in RGB mode**. With the palette you can:

* Get a `TvColor` from a palette index.
* Get all entries of the palette (their index, and the `TvColor` associated with its RGB values).
* Add a new color to the palette. This can only be done if the palette is not fixed (its `IsFreezed` property is `false`).
* Change a color of the palette (i. e. previously index 120 was dark brown, now is light orange). This can only be done if the palette is not fixed (its `IsFreezed` property is `false`)
* Get the color mode of the application

Also, it is possible to configure color converters: a color converter converts a RGB color (created with `TvColor.FromRGB`) to a palettized color of the current palette. If configured it allows use RGB colors in a palettized color mode. However final color is just an approximation and there is a penalty hit.

> Note that when you are in `Direct` color mode, you are not using a palette at all. However TVision2 provides you with a palette object, holding the current terminal palette, so you can retrieve colors from this palette and display them in the exact same way yo would do it using the `Palettized` color mode.

## Fixed color mode

Your application will run in fixed color mode in Windows if not using the new _virtual terminal_. Only Windows 10 can use the _virtual terminal_, so if in Windows 7 your application will be in fixed color mode always. The fixed color mode of Windows is the "4 bits ANSI" (16 colors). In Linux your application will run in the fixed color mode, based on the terminal of the host.

There are two main fixed color modes:

* Ansi 4 bits (16 colors)
* Ansi 3 bits (8 colors)

In this color modes, the value of a color is a ANSI standard value. For example, the red is always 4, so you can create the color red by using `TvColor.FromRaw(4)`. However as the colors are fixed, TVision2 offers you a set of predefined `TvColor` values, so you can use `TvColor.Red` instead. There is also a utility class (`TvColorNames`) that gives some methods for converting between color names and its integer value in the ANSI standard.

You can notice that there are only 8 predefined colors in `TvColor` (and `TvColorNames`): the 8 colors defined in ANSI 3 bits per color. What happens if using ANSI 4 bits (like Windows)? In this mode, the extra 8 colors are the brighten value of the 8 standard ones (i. e. you have red that is 4 and bright red wich is 12). However, you **should not create a bright red color**. Instead you should use the standard red with the bright attributte applied on it. We will see attributes later when discussing how to do character output.

## Direct color mode

This is the easiest color mode to deal with, and it is available in Windows 10 and in Linux (using the Ansi console driver). In this color mode, you create the colors by using `TvColor.FromRGB`. Although there is no really a palette, TVision2 offers you a palette, that you can setup and fill when starting the app. With this palette you can still use the `TvColor.FromPaletteIndex` for the palettized colors. Of course, you can choose to **not load** a palette, and in this case the palette provided to you will contain only the ANSI colors (so, yes: `TvColor.Red` works in direct color mode, and it is still red).

To enable this color mode you must call `EnableTrueColor()` when configuring the terminal. Otherwise TVision2 will default to palettized color mode.

