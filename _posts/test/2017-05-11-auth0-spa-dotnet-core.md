---
layout: article
title:  Auth0 with DotNet Core & React 
tags: [auth0 openid-connect react typescript C# dotnet-core]
categories: articles
comments: true
excerpt: Authentication on a React SPA with Auth0 and DotNet Core 
image:
  teaser: typescript/ts-400x250.png
ads: true
---

## React's Higher Order Components in TypeScript

*May 11, 2017*

I think my least favourite part of web development is Authentication and Authorization.  It's always one
of the first things you have to do and it's generally tedious and obscure, and if you screw it up your company
may end up on CNN.  Reading up on cryptography last year made me a little paranoid, so I now try to offload
as much of this stuff as I can to experts---in this case, [auth0](https://auth0.com/).

The goal of this example app is to allow a user to sign up/log in in a React Single Page App, and 
power the SPA with a DotNet Core API.  We'll also implement some simple role-based authentication---all without even
bothering with a database.

So there are several distinct parts here:

- A React login dialogue
- Some React/Redux infrastructure to handle JWT tokens
- Some server-side configuration
- Some configuration on Auth0

## Auth0

I have been using Auth0 for a few months and the breadth of their product always impresses me.  If there's a 
framework somewhere and an authentication standard, there's probably a blog post about it.  However, I had
 some trouble figuring out how this all works because there's just too much documentation, and all the pieces 
 are spread out around the web.  
 
 Oh, yeah, and I'd recommend *against* reading anything about the the OpenID Connect standard until after 
 you've already implemented something.  It really is difficult to understand, because most terms are very
 airy and refer to rather ambiguous concepts.  It's better to barge ahead.
 
 So first off, you can set up an Auth0 account.  The setup is wizard-y---just choose whatever defaults seem
 appropriate.  Click on "APIs" and if you're older than 22, engage in some cringe-suppression while you 
 click the "Let's Rock" button.  At this point, you should have something called "Auth0 Management API". 
 
 Next, click on "Clients".   In the ambiguous terminolog in the World of Authentication, a "Client" refers 
 to the software that accesses Auth0---in our case this will be our TypeScript SPA.  Click on "Create Client", 
then "Single Page Web Applications".  Ok, we're done here for now.

# DotNet Core

Set up the WebApi stuff the way you usually do.  If you don't have a preferred setup, I always a) create
a blank solution 2) create "src" and "test" directories, then 3) create a WebApi project in `src` called `Api`.

# React

Set up a TypeScript application the usual way, with [create-react-app-typescript](https://github.com/wmonk/create-react-app-typescript):
I'd recommend `cd`-ing to `src` and typing this:

<pre>
    > npm install -g create-react-app

    > create-react-app my-app --scripts-version=react-scripts-ts
    > cd my-app/
    > npm start
</pre>

Just to be clear, *don't* screw around with trying to configure webpack to get TypeScript going.

# Auth0-Lock

Ok, the first coding part will be to get the user logging in.  This is probably the most complex part, so we'll
get it out of the way first.




