---
layout: article
title: JWT Authentication in a DotNet Core Single Page App 
categories: articles
tags: [dotnet-core webapi spa jwt identityserver4 react]
comments: true
excerpt: Configuring a dotnet core webapi authentication with JWT
image: 
  teaser: kinect/first_try_thumb_400x250.png
ads: true
---

One way of authenticating a user for a single page application running on .Net Core
is to issue JWT tokens and then use them to validate the user and that user's roles.

[IdentityServer4](http://docs.identityserver.io/en/release/) gives you a large number
of options to support several different authentication flows.  Each flow is a 
[**grant type**](http://docs.identityserver.io/en/release/topics/grant_types.html)
