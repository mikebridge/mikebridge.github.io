---
layout: article
title: JWT Authentication in a DotNet Core Single Page App 
categories: articles
tags: [dotnet-core webapi spa jwt identityserver4 react]
comments: true
excerpt: Configuring a dotnet core webapi authentication with JWT
image: 
  teaser: signalr/token_400x250.png
ads: true
---




One way of authenticating a user for a single page application running on .Net Core
is to issue JWT tokens via [OpenID](http://openid.net/connect/) and then use them to validate 
the user and that user's roles in [SignalR](https://www.asp.net/signalr).

[IdentityServer4](http://docs.identityserver.io/en/release/) gives you a large number
of options to support several different authentication flows.  Each flow is a 
[**grant type**](http://docs.identityserver.io/en/release/topics/grant_types.html)

The concepts are many, and confusing.  

Claims
Principal
Identity
Auth Token
Identity Token
OpenID, OAuth2
Roles
Asymmetric Keys
Symmetric Keys
Flows
Implicit Flow - In other flows there may be a "client secret" that the client passes to IdentityServer to 
verify the identity of the Client application.  But the Implicit flow does not require a client secret---instead
it uses the Redirect URL that you pass in the client call.  If the domain you pass in matches one of the `RedirectUris`
that are listed in your Client configuration then your client is valid.

see: https://leastprivilege.com/2016/12/01/new-in-identityserver4-resource-based-configuration/
User

Why both an Auth and Identity token?  This was done to maintain backwards compatility.  [See this](http://www.thread-safe.com/2011/11/openid-connect-tale-of-two-tokens.html)


## Set up IdentityServer4

To start using IdentityServer4, I'd suggest downloading one of the 
[examples](https://github.com/IdentityServer/IdentityServer4.Samples) and using that 
as a starting point.  The suggested flow for an SPA application is _Implicit_.  If 
you're using ASP.Net Identity you may want to start with 
[QuickStart 6](https://github.com/IdentityServer/IdentityServer4.Samples/tree/dev/Quickstarts/6_AspNetIdentity),
but for this example we'll use [QuickStart 7](https://github.com/IdentityServer/IdentityServer4.Samples/tree/dev/Quickstarts/7_JavaScriptClient)
which will allow us later to implement our own backing store.

All you need from the example is 
[the IdentityServer project](https://github.com/IdentityServer/IdentityServer4.Samples/tree/dev/Quickstarts/7_JavaScriptClient/src/QuickstartIdentityServer)---
we can ignore the rest of the solution for now.

IMPORTANT NOTE: To get this to work, you need to upgrade to at least IdentityServer4 version 1.0.2.  
I'm using 1.1.1.
([Access tokens required special configuration before](http://stackoverflow.com/questions/41664604/claims-for-identityserver4-user-not-included-in-jwt-and-not-sent-to-web-api))

<pre>
  "dependencies": {
    "IdentityServer4": "1.1.1"
  }
</pre>

In Visual Studio, you can create an empty Solution and add this new project to it.

First thing, we'll need to figure out the [resources](http://docs.identityserver.io/en/release/configuration/resources.html) 
that you want to protect.  The terminology on this is exceptionally confusing---and [it changed recently]
We want to think about _Identity Resources_, because in our application we want the user's Id and Name,
and we also want to think about _API Resources_.  

For our application we'll use the name and id of the user, so we define it in `Config.cs` like this:


    public static IEnumerable<IdentityResource> GetIdentityResources()
            {
                var userProfile = new IdentityResource
                {
                    Name = "profile.user",
                    DisplayName = "User profile",
                    UserClaims = new[] { "name" }
                };
                return new List<IdentityResource>
                {
                    new IdentityResources.OpenId(),
                    new IdentityResources.Profile(),
                    userProfile
                };
            }

These claims will show up in the Identity Token.  

Next, we want to protect our Chat API with an _API Resource_.  In our imaginary program, we might have other, independent
APIs.  We'll name our resource "chatapi" in `Config.cs`:

        public static IEnumerable<ApiResource> GetApiResources()
        {
            return new List<ApiResource>
            {
                new ApiResource("chatapi", "Chat API")
            };
        }

Our flow is going to be `Implicit`.  So we need to tell IdentityServer about the `Client`
application that is allowed to authenticate:

TODO: Paste the code

The oidc configuration in the JavaScript client has to match with our Client configuration
in IdentityServer4.  Here's how I set it up:

```javascript
// todo: include the js client here

```

    new Client
                    {
                        ClientId = "js.implicit",
                        ClientName = "JavaScript Client",
                        AllowedGrantTypes = GrantTypes.Implicit,
                        AllowAccessTokensViaBrowser = true,
    
                        RedirectUris =
                        {
                            UrlUtils.UrlCombine(baseUrl, "/callback")
                        },
                        PostLogoutRedirectUris =
                        {
                            baseUrl
                        },
                        AllowedCorsOrigins =
                        {
                            baseUrl
                        },
    
                        AllowedScopes =
                        {
                            IdentityServerConstants.StandardScopes.OpenId,
                            IdentityServerConstants.StandardScopes.Profile,
                            IdentityServerConstants.StandardScopes.Email,
                            
                            "api"
                            //"chatapi"
                        },
                        RequireConsent = false,
                        AllowRememberConsent = false,                    
                        AlwaysSendClientClaims = true,
                        AlwaysIncludeUserClaimsInIdToken = true,
                        AccessTokenType = AccessTokenType.Jwt,
                       // UpdateAccessTokenClaimsOnRefresh = 
                    }
    
                };

## The JavaScript client

The [OIDC client](https://github.com/IdentityModel/oidc-client-js) handles the communication with the 
IdentityServer to help you out with session and token management.

In our example application, we launch the login process

<pre>
 const oidcManager = new Oidc.UserManager(oidcImplicitSettings);
</pre> 
