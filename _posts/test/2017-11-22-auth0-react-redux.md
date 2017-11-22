---
layout: article
title: Auth0 with .Net Core WebApi and React and Redux
tags: [auth0 dotnet core react redux]
categories: articles
comments: true
excerpt: Quick setup for DotNet Core and React
image:
  teaser: teamcity/wormhole-400x250.png
ads: true
---

*November 22, 2017*

>
> This is part of a series of posts on **.Net Core Core**, **TypeScript** and **React**.
>

Using [JWT](https://jwt.io/) can simplify several ...

This post assumes knowledge of what a Json Web Token is, and
some understanding of React/Redux.  The goal is to authenticate a user
in a .Net Core API, but this is the simplest part.

Sign up for the free evaluation of [Auth0](https://auth0.com) and
set up a client:

<figure>
 	<img src="/images/auth0/auth0signup.png">
 	<figcaption>Client Signup</figcaption>
</figure> 

<figure>
 	<img src="/images/auth0/auth0signup2.png">
 	<figcaption>Client Signup</figcaption>
</figure> 

You'll need to configure a unique domain---I just called mine "mikebridge":

<figure>
 	<img src="/images/auth0/auth0client_setup1.png">
 	<figcaption>Client Setup #1</figcaption>
</figure> 

And we'll need these URLs for our test server later.

<figure>
 	<img src="/images/auth0/auth0client_setup.png">
 	<figcaption>Client Setup #2</figcaption>
</figure> 


Also set the "Allowed Origins (CORS)" field to `http://localhost:3000` (not pictured).

Then, create an "Auth0 API".  The Client needs something to access,
so create an API in the Auth0 management console:

<figure>
 	<img src="/images/auth0/auth0_signupapi.png">
 	<figcaption>Set up an API</figcaption>
</figure> 



We want the JavaScript part of our application to do the following:

- Detect when a user is not logged in (or their token has expired)
- Redirect these users to Auth0 to log in there
- Accept logged-in users back to the site, and save the information.

Our .Net API just needs to validate the token and extract the user's
information from it.

## React / Redux

To start with, we'll set up redux first.  We'll be able to check
whether we're logged in or not by checking first if the the
`auth0` data has been populated, and then whether the JWT has expired
or not.

I've set up an app within a .Net Solution using the [TypeScript fork of Create React App](https://github.com/wmonk/create-react-app-typescript).  I
use `yarn` on the command line to compile and run the front end via node
rather than Visual Studio.

So step 1 is to check whether we're authenticated, and redirect
unauthenticated users to our Login page on Auth0.  This is a
good place for a [Higher Order Component](articles/getting-started-typescript-react-4/) wrapper.  If, for example, we want to create a React `Chat`
component, but ensure that a user is logged in first, we want it to
look something like this:

<pre>

   class Chat extends React.Component<IChatProps> {
       public render(): JSX.Element {
           return (
               <div>Hello world.</div>
           );
       }
   }
   export default loginWhenUnauthorized(Chat);
</pre>

That effectively separates the login/auth logic from our Chat component.

The wrapper will check whether the current user has valid login information
in redux while it's mounting, and either display the wrapped component
or else set a flag in redux that indicates that a login is required.  My
initial version of this used Middleware to capture a "login request"
action, but this turned out to be messy---having a "loginNeeded" state
variable in redux is much cleaner.  I now have a component `Auth0Login`
which watches this loginNeeded variable and initiates the login
process.

The `auth` value in our redux store is going to store the JWT,
an expiry date and this loginNeeded value:

<pre>
interface IAuthStore {
    jwt?: string;
    dateExpires?: Date;
    loginRequested: boolean;
}
</pre>

In this example I'm using `react-router's` `withRouter` wrapper
to inject the browser url (i.e. window.location).

TODO: Post gist for loginWhenUnauthorized and auth0Reducers.


If things went well, navigating to the `http://localhost:3000/chat` url
will now launch the default Auth0 login popup:

<figure>
        <img src="/images/auth0/auth0_login.png">
 	<figcaption>The default Auth0 login popup</figcaption>
</figure> 

It won't work yet--we still need to process the result in the callback function.

If you try to log in you'll see that you get the following
variables back in the URL:

access_token
scope
expires_in
token_type
state=bnVsbA%3D%3D
id_token




