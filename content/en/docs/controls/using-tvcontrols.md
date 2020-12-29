---
title: "Using TvControls"
linkTitle: "Using TvControls"
weight: 1
description: >
  How to use TvControls library
---

To use TvControls **you must include the reference to it Nuget package** and configure your application to use TvControls:

```
var builder = new HostBuilder();
builder.UseTvision2(setup => setup
   .AddTvControls(sk => sk.AddMcStyles()));
```

The `AddTvControls` method adds TvControls to your library. The method accepts an optional delegate of type `Action<ISkinManagerBuilder>` to configure the theme used by all the controls of the application. 

## The Hello World using controls

