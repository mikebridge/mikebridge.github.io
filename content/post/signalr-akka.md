+++
title = "SignalR and Akka.NET on DotNet Core"
description = "Configure SignalR and Akka.NET on DotNet Core"
tags = ["signalr", "akka.net", "ASP.Net Core", "es2016", "react", "owin", "cqrs"]
date = "2017-02-13"
aliases = ["articles/signalr-akka"]
+++

>
> Note: this post is out-of-date---Akka.NET [now runs on Dotnet Core](https://petabridge.com/blog/akkdotnet-v13-dotnetcore/)
>

Currently there is no [DotNet Core runtime](https://dotnet.github.io/) support 
for [Akka.NET](http://www.aaronstannard.com/dotnetcore-boil-ocean/) 
or SignalR.  I was about resign myself to the idea of waiting for another year
or so to try them out together in the new environment, but I randomly came across a 
blog that mentions that *you can run non-core libraries on ASP.Net Core*
— if you target _.Net 4.6.1_.  [Good to know](https://jonhilton.net/2016/09/07/using-asp-net-core-against-net-4-6/).

I'm using [Visual Studio 2015 Update 3](https://www.visualstudio.com/en-us/news/releasenotes/vs2015-update3-vs)
and the [DotNet Core Tools v. 1.1](https://www.microsoft.com/net/download/core#/current).

_**TL;DR** the code for this example is at [https://github.com/mikebridge/AkkaSignalR](https://github.com/mikebridge/AkkaSignalR)_.

This demo has two moving parts.  There's a React.js front end running on Node, and a back end 
with the simplest Actor-based service I can think of—an "Echo Service" which just echoes back 
whatever it receives.

{{<figure src="/images/akka/akka_signalr.png" caption="The Demo app." caption-effect="fade" caption-position="bottom">}}

# Configure a new SignalR Application

To create a C# application from scratch, create an empty solution in Visual Studio and then
add an empty DotNet core Web Application project inside that.

Since we can't target for `netcoreapp1.0` yet, open
[`package.json`](https://github.com/mikebridge/AkkaSignalR/blob/master/src/EchoAPI/project.json#L24-L26) in the project and replace `netcoreapp1.0` 
under `frameworks` with `net461`:

```json
    "frameworks": {
        "net461": {
        }
    }
```
 
You'll also need to remove these lines from `dependencies`, since `Microsoft.NETCore.App` is incompatible with 4.6.1:  

<pre>
<strike>
    "Microsoft.NETCore.App": {
        "version": "1.0.1",
        "type": "platform"
    }
</strike>
</pre>    

Add these dependencies to [`package.json`](https://github.com/mikebridge/AkkaSignalR/blob/master/src/EchoAPI/project.json#L9-L16) (or get them from NuGet):

<pre>
    "Microsoft.AspNetCore.Owin": "1.1.0",
    "Microsoft.AspNetCore.Cors": "1.1.0",   
    "Microsoft.AspNet.SignalR": "2.2.1",    
    "Akka": "1.1.3",
    "Akka.Remote": "1.1.3"
</pre>

... and execute `dotnet restore` or right-click on your project's "References" and 
select "Restore Packages".

### Configure Startup.cs

Since SignalR has an old-style setup method, we'll have to [bridge the gap between the old IAppBuilder and the new
IApplicationBuilder-based configuration](https://blogs.msdn.microsoft.com/webdev/2014/11/14/katana-asp-net-5-and-bridging-the-gap/).
I adapted the official Microsoft
 `UseAppBuilder` extension function [here](https://github.com/mikebridge/AkkaSignalR/blob/master/src/EchoAPI/KatanaIApplicationBuilderExtensions.cs).
This is just a temporary solution until the new pipeline is supported in SignalR.

[`Startup.cs`](https://github.com/mikebridge/AkkaSignalR/blob/master/src/EchoAPI/Startup.cs) will look something like this:

```csharp
public class Startup
{ 
   public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
   {
       services.AddCors(options =>
       {
           options.AddPolicy("default", policy =>
           {
                policy.WithOrigins("http://localhost:3000")
                    .AllowAnyHeader()
                    .AllowAnyMethod();
           });
       });
   }
   
   public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
   {
       app.UseCors(policy =>
       {
            policy.WithOrigins("http://localhost:3000");
            policy.AllowAnyHeader();
            policy.AllowAnyMethod();
            policy.AllowCredentials();
       });
   
   
       var actorSystem = ActorSystem.Create("SignalRChatAPI");

       var echoActor = actorSystem.ActorOf(Props.Create(() => new SignalREchoActor()), "echoActor");

       app.UseAppBuilder(appBuilder => appBuilder.Use((ctx, next1) =>
       {
           // make the actor system available via the owin environment
           ctx.Environment["akka.actorsystem"] = actorSystem;
           return next1();
       }).MapSignalR());
   }
}

```

Note that CORS is configured to allow connections from our Node server, which
will be running locally on port 3000.

### Join SignalR to Akka

You can either connect Akka to SignalR via a `PersistentConnection` or 
using `Hubs`.  Either works fine, but I preferred the higher-level
abstraction of using Hubs---plus it seems easier to hook it into 
authentication. 

Here's the single message type that this system supports:

```csharp
using Akka.Actor;
using EchoAPI.Hub;
using EchoAPI.Messages;
using Microsoft.AspNet.SignalR;

namespace EchoAPI.Actors
{
    public class SignalREchoActor : TypedActor,
        IHandle<EchoRequest>
    {

        private IHubContext _context;

        protected override void PreStart()
        {
            _context = GlobalHost.ConnectionManager.GetHubContext<EchoHub>();
        }

        public void Handle(EchoRequest message)
        {          
            // send the message back to the client.
            _context.Clients.All.echoMessage($"ECHO: {message.Message}");
        }
    }
}
```

The [echo actor](https://github.com/mikebridge/AkkaSignalR/blob/master/src/EchoAPI/Actors/SignalREchoActor.cs) 
handles the `EchoRequest` messages it receives and sends them back to all the current JavaScript clients. 
It finds those clients via the `IHubContext`, which it locates using
`GlobalHost.ConnectionManager.GetHubContext`:

```csharp
using Akka.Actor;
using EchoAPI.Hub;
using EchoAPI.Messages;
using Microsoft.AspNet.SignalR;

namespace EchoAPI.Actors
{
    public class SignalREchoActor : TypedActor,
        IHandle<EchoRequest>
    {

        private IHubContext _context;

        protected override void PreStart()
        {
            _context = GlobalHost.ConnectionManager.GetHubContext<EchoHub>();
        }

        public void Handle(EchoRequest message)
        {          
            // send the message back to the client.
            _context.Clients.All.echoMessage($"ECHO: {message.Message}");
        }
    }
}
```

The Hub passes along client requests to the Echo Actor, and it finds the actor via the ActorSystem.  I'm
getting the ActorSystem from the Owin environment variable `Context.Request.Environment["akka.actorsystem"]` 
that we configured in `Startup.cs`, but the [Offical Documentation](https://petabridge.com/blog/akkadotnet-aspnet/) says
that you can use a global variable to find the Actor instead.

```csharp
using System;
using System.Diagnostics.CodeAnalysis;
using Akka.Actor;
using EchoAPI.Messages;
using Microsoft.AspNet.SignalR.Owin;

namespace EchoAPI.Hub
{
    [SuppressMessage("ReSharper", "ClassNeverInstantiated.Global")]
    public class EchoHub : Microsoft.AspNet.SignalR.Hub
    {

        public void SendMessage(String message)
        {
            var actorSystem = FindActorSystem();
            var echoActorRef = actorSystem.ActorSelection("/user/echoActor");
            echoActorRef.Tell(new EchoRequest(message));

        }

        private ActorSystem FindActorSystem()
        {
            var ctx = Context.Request as ServerRequest;
            if (ctx == null)
            {
                throw new Exception("The context was not initialized");
            }
            var actorSystem = ctx.Environment["akka.actorsystem"] as ActorSystem;
            if (actorSystem == null)
            {
                throw new Exception("The ActorSystem was not initialized");
            }
            return actorSystem;
        }
    }
}
```

You will need to adjust the port that the API runs on in [Program.cs](https://github.com/mikebridge/AkkaSignalR/blob/master/src/EchoAPI/Program.cs#L14).
 
```csharp
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
             .UseKestrel()
             .UseContentRoot(Directory.GetCurrentDirectory())
             .UseUrls("http://localhost:5001") // add this to run on port 5001
             .UseIISIntegration()
             .UseStartup<Startup>()
             .Build();
 
    host.Run();
}
```

### Miscellaneous Configuration 

To be able to launch the server from Visual Studio,

- Uncheck `Properties->Debug->Launch URL`
- Set `Properties->Debug->WebServer Settings->App URL` to `http://localhost:5001/`.

Also, if you want capitalized C# method names to be camelCased for the JavaScript client, 
you'll need to do something like 
[this](http://stackoverflow.com/questions/30005575/signalr-use-camel-case#answer-30019100).
(I'm using [this implementation](https://github.com/mikebridge/AkkaSignalR/blob/master/src/EchoAPI/SignalRContractResolver.cs)).

### Next Steps

Now that we have SignalR and Akka.NET working together with Owin Middleware, 
it will be relatively easy to add JWT Token authentication.
 
Also, the current [React code](https://github.com/mikebridge/AkkaSignalR/blob/master/src/Web/src/Echo.js)
is rather ugly.  It would also be nice to have a better separation
 of concerns so that connecting, messaging and view handling
 are more modular.
  
I'll add some notes on both these in upcoming posts.


