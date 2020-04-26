---
title: "What is Tvision2"
linkTitle: "What is Tvision2"
weight: 1
description: >
  What is Tvision2, why was built and what you can do with it.
---

TVision2 is a **netstandard** library created to help you building cross platform console applications with a rich [TUI](https://en.wikipedia.org/wiki/Text-based_user_interface).

## Why TVision2 was created?

This is a great question. Currently there are some other awesome netstandard libraries out there that have more or less the same purpose that Tvision2:

* [Gui.cs](https://github.com/migueldeicaza/gui.cs): This is probably the most known project. It's created and mantained by [Miguel De Icaza](https://twitter.com/migueldeicaza). Gui.cs acted as a inspiration from TVision2 and some of the Miguel's code (like Mono.curses) is currently used in TVision2.Its maturity state is more advanced than the TVision2 one. However TVision2 offers new capabalities and characteristics that outerperform Gui.cs

## What makes TVision2 special?

Of coure I am highly biased here, but TVision2 was designed since the beginning with the _pay only what do you use_ rule. The core of TVision2 offers you a rich set of mechanisms to build console applications using a game loop approach. Core do not have the concepts of event-driven actions and no "controls" like textboxes or lists are provided. Based on the services provided by core, TVision2.Controls provide a set of controls (like textboxes, lists, dropdowns, etc). You can choose to use TVision2.Controls or not, based on what do you need. If you are building an "application" probably would want to use controls, but if you are building a console-based game, maybe you wan't. However the decision is up to you. Same happens with the Layout mechanism. Core only offers absolute positioning, but you can use TVision2.Layouts to get relative layouts like stack panels, grids and so on. You can use Layouts with or without controls and vice-versa.

Also TVision2 supports not only 256 colors but also true color. It adapts to the characteristics of OS:

* In Windows 7 the Windows API console is used and applications are limited to 16 colors. Using some console manager like ConEmu is possible to run the application in 256 colors.
* In Windows 10 the new VT enabled console API is used, and applications can have 256 colors or true color.
* In Linux, application can choose to use NCurses (limited to 256 colors) or use the new VT escape codes to have true color.

TVision2 core offers an API that tries to abstract as mucha as possible the current color model used, while giving full functionality to the application developer. TVision2.Controls extends this API to offer Skins that allow the developer to define the colors of each control based on the color model used (16 colors, 256 colors, etc...).
