---
title: "Colors"
linkTitle: "Colors"
weight: 3
description: >
  Working with colors
---

Dealing with colors is complex, because there are important differences between *NIX and Windows. The API for colors is built upon the `TvColor` type. The `TvColor` abstracts three APIs in one:

1. The RGB API, used when working in direct (full color) mode
2. The palette API, used when working in palettized modes (like 256 colors)
3. The index API, used when working in fixed-color modes (like 16 colors)

If you are in RGB color mode, can create a `TvColor` by calling `TvColor.FromRGB` method. If you are in palettized mode, you can use `TvColor.FromPaletteIndex` and if you are in fixed-color mode you can use `TvColor.FromRaw`. Let's discuss all these methods a little further.

## Fixed color mode

Your application will run in fixed color mode in Windows if not using the new _virtual terminal_. Only Windows 10 can use the _virtual terminal_, so if in Windows 7 your application will be in fixed color mode always. The fixed color mode of Windows is the "4 bits ANSI" (16 colors). In Linux your application will run in the fixed color mode, based on the terminal of the host.

There are two main fixed color modes:

* Ansi 4 bits (16 colors)
* Ansi 3 bits (8 colors)

In this color modes, the value of a color is a ANSI standard value. For example, the red is always 4, so you can create the color red by using `TvColor.FromRaw(4)`. However as the colors are fixed, TVision2 offers you a set of predefined `TvColor` values, so you can use `TvColor.Red` instead. There is also a utility class (`TvColorNames`) that gives some methods for converting between color names and its integer value in the ANSI standard.

You can notice that there are only 8 predefined colors in `TvColor` (and `TvColorNames`): the 8 colors defined in ANSI 3 bits per color. What happens if using ANSI 4 bits (like Windows)? In this mode, the extra 8 colors are the brighten value of the 8 standard ones (i. e. you have red that is 4 and bright red wich is 12). However, you **should not create a bright red color**. Instead you should use the standard red with the bright attributte applied on it. We will see attributes later when discussing how to do character output.

## Palettized color mode

In **Windows the palettized color mode is NEVER used**, but in Linux is the most common one. Most known terminals like `xterm-256` works this way. In this mode, a color is simply an integer with the index of the color in the palette. So, **based on your palette** the color index 120 can be red, lime or turquoise. Fortunately **most palettes have the same first 16 values like the Ansi 4 bits colors**. If this is the case, you can use a `TvColor` like `TvColor.Red` in a palettized application and the result color will be Red. If the first colors of the palette are not like the ANSI standard, you can sill use `TvColor.Red` but there is no guarantee that the real color will be red!

In a palettized color mode, the palette can be "fixed" or "configurable". In a "configurable" palette you can choose the colors you want to use from a boarder range of colors (like choosing 256 colors from a range of millions). In a "fixed" palette, the palette is, well, fixed by the terminal, and can't be changed. When working in a palettized mode the main question is "how I can know what color is the color index 120?".

TVision2 offers the `IPalette` interface for this question. And **to simplify your application development, TVision2 will ALWAYS offer you a palette, even though your application is running in a fixed color mode or in RGB mode**. With the palette you can:

* Get a `TvColor` from a palette index.
* Get all entries of the palette (their index, and the `TvColor` associated)
* Add a new color to the palette. This can only be done if the palette is not fixed (its `IsFreezed` property is `false`).
* Change a color of the palette (i. e. previously index 120 was dark brown, now is light orange). This can only be done if the palette is not fixed (its `IsFreezed` property is `false`)
* Get the color mode of the application

Also, it is possible to configure color converters: a color converter converts a RGB color (created with `TvColor.FromRGB`) to a palettized color of the current palette. 

## Direct color mode

This is the easiest color mode to deal with, and it is available in Windows 10 and in Linux (but using an experimental console driver). In this color mode, you create the colors by using `TvColor.FromRGB`. Although there is no really a palette, TVision2 offers you a palette, that you can setup and fill when starting the app. With this palette you can still use the `TvColor.FromPaletteIndex` for the palettized colors. Of course, you can choose to **not load** a palette, and in this case the palette provided to you will contain only the ANSI colors (so, yes: `TvColor.Red` works in direct color mode, and it is still red).

