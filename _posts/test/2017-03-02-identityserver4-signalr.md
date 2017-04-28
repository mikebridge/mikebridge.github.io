---
layout: article
title: JWT Tokens, SignalR and Single Page Applications 
categories: articles
tags: [dotnet-core spa jwt identityserver4 react signalr webapi openid-connect]
comments: true
excerpt: Configuring SignalR to authenticate with IdentityServer4 and OpenID Connect
image: 
  teaser: signalr/token_400x250.png
ads: true
---

A new-ish alternative to session-based cookies that's well-suited to single page apps
is **token-based authentication**.  

There are many SaaS services such as [Auth0](https://auth0.com/), [Stormpath](https://stormpath.com/) and
[Login Radius](https://www.loginradius.com/) that are pretty easy to set up.  If you can use one of
those in your organization, you should---it will save you a lot of time.  But if you're stuck hosting
your data yourself you will need to look at a product like [IdentityServer4](https://identityserver.io/) or 
[ForgeRock](https://www.forgerock.com/).  Today's blog entry is the result of my evaluation of 
integrating OpenID Connect and SignalR using [IdentityServer4](https://identityserver.io/).  

_**TL;DR** The code for this example is at 
[https://github.com/mikebridge/IdentityServer4SignalR/](https://github.com/mikebridge/IdentityServer4SignalR/)_.


> **Edit Apr 28, 2017:**
>
> Since writing this post I've switched from IdentityServer to [Auth0](https://auth0.com/).  It
> has quite a few more features, requires less setup, and has friendly support.

<figure>
 	<img src="/images/signalr/whosonfirst.png">
 	<figcaption>A React Chat App.</figcaption>
</figure>

Before you get started, you should realize that implementing IdentityServer4 requires a lot 
of coding.  The spec is rather confusing, the documentation is voluminous and the project 
maintainers don't do much hand-holding, so the learning curve is steep.  Caveat Emptor! 

## The Basic Idea

The basic idea is this: we send a user from our JavaScript application to a web server 
running IdentityServer4.  IdentityServer4 hands out two tokens to the user if he can 
prove his identity somehow (maybe via social media, maybe via password), and the user then 
sends one of the tokens he receives to our API—in this demo, a very simple [SignalR](https://www.asp.net/signalr) Chat 
App API.  Our API then authenticates that token to determine whether the user should have access 
to a particular API call.

This sounds simple, but there's a lot of configuration to do to get there.  You may want to check
out the [previous blog post](/articles/signalr-akka) about configuring SignalR with DotNet Core.

**[JWT](https://jwt.io/introduction/)s**, or **JSON Web Tokens** is what connects everything 
together.  JWT tokens can contain information about the identity of someone (like 
names, email addresses, etc.) as well as other information, such as permissions or roles.  Bits 
of information contained in the payload of a JWT token are called "claims"—e.g. I claim that 
my name is "Mike" and I claim to have "admin" access, the issuing server claims that the token 
expires at midnight, and so on.

Tokens are signed by IdentityServer4 and verified by any app that has access to the
right information.  Because asymmetric keys have the advantage that there is no shared secret to protect 
among all the clients that need to authenticate users, we'll be using them in this demo.
So in our case, a IdentityServer has a private key, and any client that wishes to identify the
token will have the corresponding public 
key.  (The installation of test keys is described [in the example project](https://github.com/mikebridge/IdentityServer4SignalR/#create-and-install-asymmetric-keys).)

### IdentityServer4

[IdentityServer4](http://docs.identityserver.io/en/release/) gives you a large number
of options and supports several different authentication "flows", depending on the type of 
[**client**](http://docs.identityserver.io/en/release/reference/client.html). A client is the 
application accessing IdentityServer---either a native application, a traditional web 
application or a JavaScript-based application.  Each flow is a 
[**grant type**](http://docs.identityserver.io/en/release/topics/grant_types.html).  Since
we're using React, we'll use the OpenID Connect Implicit grant type, which was developed
for that kind of client application.  In other grant types, there is client secret that is 
*explicitly* passed to identify a client, but since we're passing the information in 
the clear via javascript, the client identifies itself *implicitly* by passing a 
"redirect URI" to IdentityServer4 (which tells where to redirect
the user after the authentication procedure is complete), and IdentityServer4 will check
that URI against its list of allowed URIs.  There's a good explanation of this 
[here](https://leastprivilege.com/2016/12/01/new-in-identityserver4-resource-based-configuration/). 

One confusing thing: OpenID Connect generates _two_ JWT tokens with overlapping information: 
an **identity token** and an **access token**.  This is apparently for backward-compatibility 
reasons.  Both contain your claims, but there are some slight differences between the two 
tokens---there are some extra security features with Identity Tokens (e.g. nonce validation against 
cross-site request forgery), but it will be the access tokens that get passed to our SignalR 
server. [Here's a more complete explanation](http://www.thread-safe.com/2011/11/openid-connect-tale-of-two-tokens.html).
  
## Set up IdentityServer4

To start using IdentityServer4, you should download one of the 
[examples](https://github.com/IdentityServer/IdentityServer4.Samples) and use that 
as a starting point.  As I mentioned, the suggested flow for an SPA application is _Implicit_, so if 
you're using ASP.Net Identity you may want to start with 
[QuickStart 6](https://github.com/IdentityServer/IdentityServer4.Samples/tree/dev/Quickstarts/6_AspNetIdentity).
But for this example we'll use [QuickStart 7](https://github.com/IdentityServer/IdentityServer4.Samples/tree/dev/Quickstarts/7_JavaScriptClient)
which will allow us to implement our own backing store later.

All you need from the example is 
[the IdentityServer project](https://github.com/IdentityServer/IdentityServer4.Samples/tree/dev/Quickstarts/7_JavaScriptClient/src/QuickstartIdentityServer)---
we can ignore the rest of the solution for now.

**NOTE**: You need to upgrade to at least IdentityServer4 
version 1.0.2.  I'm using 1.1.1.  Versions prior to 1.0.2 had a [bug](http://stackoverflow.com/questions/41664604/claims-for-identityserver4-user-not-included-in-jwt-and-not-sent-to-web-api) 
that prevented claims from being included in the access token.

<pre>
  "dependencies": {
    "IdentityServer4": "1.1.1"
  }
</pre>


In Visual Studio, create an empty Solution and add this new "IdentityServer" quickstart project to it.

To secure an app, you need to identify the [**resources**](http://docs.identityserver.io/en/release/configuration/resources.html) 
that you want to protect.  The terminology on this is somewhat confusing---and 
[it changed recently from "**scopes**"](https://leastprivilege.com/2016/12/01/new-in-identityserver4-resource-based-configuration/).
In this app we need _Identity Resources_---the user's Id and Name, as well as _API Resources_---the name of our API.  

For our application we'll use define the profile information we want to receive about the user in [`Config.cs`](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/IdentityServer/Config.cs#L20-L35) 
like this:

<script src="https://gist.github.com/mikebridge/aba639164fdcd09b464fc11815f53efb.js"></script>

These claims will show up in each token.

Next, we want to protect our Chat API as an _API Resource_.  We'll define our resource with the string "chatapi" in `Config.cs`:

<script src="https://gist.github.com/mikebridge/b8696002ed25b0baa59caf3b8d73dca4.js"></script>

Our flow is going to be `Implicit`.  So we need to tell IdentityServer to accept requests from our 
JavaScript `Client` application:

<script src="https://gist.github.com/mikebridge/cc98b11b4bd0370fb3f12295b90f5994.js"></script>

In a real application, there would probably be a database of users---but for this example,
we'll create an [in-memory test store](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/IdentityServer/Services/TestUserService.cs).
called `TestUserService.cs`

We'll also define our own "IProfileService".  This is not described in the IdentityServer4 
 documentation, but its purpose is to describe all the claims that will show up in both
  the identity token and the access token:
    
<script src="https://gist.github.com/mikebridge/93b6fd429dcc71cc831e35dc617b5a34.js"></script>    

Note that we're hardcoding the claims in this example.  And we've create a _scope_ called
`chatapi`, and a _role_ within that scope called `chatapi.user`.   A _scope_ is a custom set
of claims, and we're defining a particular claim as a _role_.  The role name is what we'll use later in 
the familiar [`[Authorize(Roles="chatapi.user")]`](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/ChatAPI/Hub/EchoHub.cs#L13) 
annotation in the SignalR Hub.

We can wire all this up during initialization in 
[Startup.cs](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/IdentityServer/Startup.cs#L19-L33).

<script src="https://gist.github.com/mikebridge/a0e4f9c5381563d341615f16655ac38e.js"></script>

You'll also have to [set up the key pair](https://github.com/mikebridge/IdentityServer4SignalR/#create-and-install-asymmetric-keys)
if you haven't done that already.

## Single Page App

Interaction with IdentityServer4 is done with the [oidc-client JavaScript](https://github.com/IdentityModel/oidc-client-js) 
javascript library.  [My implementation](https://github.com/mikebridge/IdentityServer4SignalR/tree/master/src/Web/src/redux-oidc) 
is React/Redux-specific so I won't go into it in too much detail.  

The oidc configuration in the JavaScript client has to match with our Client configuration
in IdentityServer4.  [Here's](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/Web/src/config/oidcConfig.ts) how I set it up:

<script src="https://gist.github.com/mikebridge/a748d73c689c9d60a7cfc4d719ccdf2e.js"></script>

The "response_type" specifies that we want both an id token and an access token, as per the OpenID
Connect specification.  The scope is "openid email profile chatapi", which reflects the
resources that we [configured in the Config.cs](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/IdentityServer/Config.cs#L20-L43)

We will launch the login process here with the 
 
<pre>
 const oidcManager = new Oidc.UserManager(oidcImplicitSettings);
 oidcManager.signinRedirect(...);
</pre> 

Then on the callback page you'll need to get the user object:

<pre>
   userManager.signinRedirectCallback().then(result => {
        userManager.getUser().then(user => {
            // save the user!
            console.log("logged in!);
            console.log(user);
        }).catch((e) => {
            console.error(e);
        });
</pre>

At the very least, you'll need to save the `user.access_token` to pass through to 
SignalR---this is the JWT access token.  You should have a look at the contents at 
[http://jwt.io](http://jwt.io).  
 
## SignalR: Client-Side
 
SignalR wasn't built to interact with token-based authentication.  Your two options
are: 1) pass the token in the query string or 2) pass the token as a parameter to
a server-side call.  The first option is a little less yucky, so we'll do that.
 
I also won't describe how to use SignalR, except to note that you'll need
to set the ".qs" on your HubConnection:

<pre>
hubConnection.qs = token ? `authtoken=${user.access_token}` : "";
</pre>

Annoyingly, you have to disconnect and reconnect if your access_token changes, e.g. 
it expires and you need to renew it.
 
(My **demo-quality** react connector is [here](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/Web/src/react-signalr/signalrConnector.tsx).)

## SignalR: Server-Side

If you've gotten this far the last thing to do is to translate the incoming request into 
something that the native DotNet core authentication system can handle.  In a WebApi you can use
the [Microsoft.AspNetCore.Authentication.JwtBearer](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/)
middleware, but to authenticate with SignalR we'll need to add an extra middleware layer.

My [ChatAPI package.json](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/ChatAPI/project.json) 
contains these dependencies:

<pre>
 "dependencies": {
    "Microsoft.AspNetCore.Diagnostics": "1.0.0",
    "Microsoft.AspNetCore.Server.IISIntegration": "1.0.0",
    "Microsoft.AspNetCore.Server.Kestrel": "1.0.1",
    "Microsoft.Extensions.Logging.Console": "1.0.0",
    "Microsoft.Extensions.Configuration.Json": "1.1.0",
    "Microsoft.Extensions.Configuration.FileExtensions": "1.1.0",

    "Microsoft.AspNetCore.Cors": "1.1.0",
    "Microsoft.AspNetCore.Mvc": "1.1.1",
    "Microsoft.AspNetCore.Mvc.Core": "1.1.1",
    "Microsoft.AspNetCore.Owin": "1.1.0",
    "Microsoft.IdentityModel.Tokens": "5.1.2",
    "Microsoft.AspNetCore.Authentication.JwtBearer": "1.1.0",
    "Microsoft.AspNet.SignalR": "2.2.1",
    ...
  }
</pre>

Setting up SignalR is a little finicky because it hasn't been updated for DotNet Core, but 
there's [more detail in this previous blog post](/articles/signalr-akka/)

We'll add in the basic configuration to [Startup.cs](https://github.com/mikebridge/IdentityServer4SignalR/blob/master/src/ChatAPI/Startup.cs):

<script src="https://gist.github.com/mikebridge/3304ea0663fb267a9ab3e7d0326cd7cd.js"></script>

We'll need to write the middleware that watches for our `authtoken` and puts it into an
`Authorization` header:

<script src="https://gist.github.com/mikebridge/75ecef2aa9529e7e419fc5848b6121ed.js"></script>

To configure the JWT authentication and wire the query-string processing middleware and
the JWT processing middleware together is verbose:
 
<script src="https://gist.github.com/mikebridge/1b6e5243716e2d3b029cee24d2994034.js"></script>
 
## SignalR

Crap, that was a lot of work to get to here.  But now that it's set up, we can use
the `Microsoft.AspNet.SignalR.Authorize` header.  (Don't get this far and
add the incompatible WebAPI `Authorize` header by accident!)

<pre>
[Authorize(Roles = "chatapi.user")]

    public class EchoHub : Microsoft.AspNet.SignalR.Hub
    {
       //...
    }
    
</pre>
   
That's it!  Next up: simplifying this whole thing with _Auth0_.


