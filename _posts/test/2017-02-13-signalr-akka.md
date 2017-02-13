---
layout: article
title: SignalR and Akka on DotNet Core
tags: [signalr, akka, dotnet-core]
categories: articles
comments: true
excerpt: Configure SignalR and Akka on DotNet Core 
ads: true
---

As of today there is not yet any [support for Akka.NET](http://www.aaronstannard.com/dotnetcore-boil-ocean/) 
or SignalR for dotnet core applications, but in the meantime you can
run both on on dotnet core on .Net 4.6.1.

I'm using [Visual Studio 2015 Update 3](https://www.visualstudio.com/en-us/news/releasenotes/vs2015-update3-vs).

# Configure the Application

First, create an empty solution in Visual Studio and then
add an empty DotNet core Web Application project inside that.

Since we can't target for `netcoreapp1.0` yet, we need to open
`package.json` in the project and replace `netcoreapp1.0` 
in the `frameworks` with `net461`:

```json
  "frameworks": {
    "net461": {
    }
  }
```
 
We'll also need to remove these lines from `dependencies`:  

```json
    "Microsoft.NETCore.App": {
      "version": "1.0.1",
      "type": "platform"
    }
```    

And add these dependencies (or get them from NuGet):

```json
    "Microsoft.AspNetCore.Owin": "1.1.0",
    "Microsoft.AspNetCore.Cors": "1.1.0",   
    "Microsoft.AspNet.SignalR": "2.2.1",    
    "Akka": "1.1.3",
    "Akka.Remote": "1.1.3"
```

... and execute `dotnet restore` or right-click on "References" and 
select "Restore Packages".

### Configure Startup.cs

For now we also need to [bridge the gap between the old IAppBuilder and the new
IApplicationBuilder-based configuration](https://blogs.msdn.microsoft.com/webdev/2014/11/14/katana-asp-net-5-and-bridging-the-gap/)
so that we can make use of the existing SignalR setup functions ([Example](https://github.com/aspnet/Entropy/blob/dev/samples/Owin.IAppBuilderBridge/KAppBuilderExtensions.cs)).
This is just a temporary solution until the new pipeline is supported:

```C#
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.Owin.Builder;
    using Owin;
    
    public static class KatanaIApplicationBuilderExtensions
        {
            
            public static IApplicationBuilder UseAppBuilder(this IApplicationBuilder app, Action<IAppBuilder> configure)
            {
                app.UseOwin(addToPipeline =>
                {
                    addToPipeline(next =>
                    {
                        var appBuilder = new AppBuilder();
                        appBuilder.Properties["builder.DefaultApp"] = next;
    
                        configure(appBuilder);
    
                        return appBuilder.Build<AppFunc>();
                    });
                });
                return app;
            }
        }
```

Then in `Startup.cs` it can be called like this:

```C#
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
       /// ...
       Configure Akka(app);
    }
    
    private void ConfigureAkka(IApplicationBuilder app)
    {
        var actorSystem = ActorSystem.Create("SignalRAkkaAPI");

        var persistentConnectionActor = actorSystem.ActorOf(Props.Create(() => new SignalRActor()), "signalRActor");

        app.UseAppBuilder(appBuilder => appBuilder.Use((ctx, next1) =>
        {
            // make the actor system available via the owin environment
            ctx.Environment["akka.actorsystem"] = actorSystem;
            return next1();
        }).MapSignalR());
    }
```

### Join SignalR to Akka

You can either connect Akka to SignalR via a `PersistentConnection` or 
using `Hubs`.  Either works fine, but I preferred the higher-level
abstraction of using Hubs---plus it seems easier to hook it into 
authentication.  THe 

### Configure CORS




### Other 

Uncheck Properties->Debug->Launch URL
Set Properties->Debug->WebServer Settings->App URL to http://localhost:5001/.
