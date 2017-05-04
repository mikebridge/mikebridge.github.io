---
layout: article
title:  TypeScript and React, Part 4
tags: [react typescript]
categories: articles
comments: true
excerpt: React Higher Order Controls in TypeScript 
image:
  teaser: typescript/ts-400x250.png
ads: true
---

The [Higher Order Component](https://facebook.github.io/react/docs/higher-order-components.html) pattern in React
addresses at least four of the [SOLID principles](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) when 
writing Components.  TypeScript helps us out by imposing some structure to help us make sure we've implemented the 
pattern correctly.

Say, for example, you want to create a component that says "Welcome {username}" and sends the user to a "User Settings"
page when that message is clicked.

The first step might be to write a simple component that displays the logged-in user's information:

<script src="https://gist.github.com/mikebridge/e6dc04180f9a1cd42410d961d1971ccf.js"></script>

And here some tests:

<script src="https://gist.github.com/mikebridge/b9bb34e9a2f60c871ab7e38e812c7c12.js"></script>

That's easy enough, but there's still some logic to implement:

1. If the user is not logged in, we want to hide this component.
2. If the user *is* logged in, we need to access his or her name, e.g. from Redux.
2. We need to trigger navigation when the component is clicked, e.g. via [react-router](https://github.com/ReactTraining/react-router).

Where should that go?  It would be easy to add all this logic right into our Welcome component by [`connect`-ing](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) it to
our Redux store and referencing the router directly.  However, there are probably a few good reasons not to do that:

- We'll probably need to personalize other components too later, so this is probably a cross-cutting concern
- The `Welcome` component is a **[Dumb/Presentational](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)**
component, and this extra logic belongs in a **Smart/Container** component.  Our simple component shouldn't
know about datastores or site navigation---it should just display data.

So really, we want to avoid adding this logic to our `Welcome` component because a) it would compromise 
its `Single Responsibility` (i.e. to display a personalized link) and b) it should be *Closed to Modification but
Open to Extension*.  HOCs can help us keep our code clean. 

Let's say we already have the user information in the Redux store under the `user` key.  A HOC wrapper could
grab that object if it exists and inject it (There's [D](https://en.wikipedia.org/wiki/Dependency_inversion_principle)) 
into the subcomponent.  I would describe this functionality as "personalization", so I'll call this HOC 
`withPersonalization`.  Here's an implementation, where the personalization is 
hardcoded (later we'll introduce Redux):

<script src="https://gist.github.com/mikebridge/93dbc41f1afc2dd4ce64f08106eefbab.js"></script>

There's some stuff that hurts the eyes here, so let's establish what we're trying to accomplish with this first 
iteration:

- When someone uses our "<Welcome />" component, they don't want to know about the name, so we want to make sure 
that the "onClick" property is visible and the "name" property is not.
- When we wrap a component in `withPersonalization`, we want to ensure that the component is aware of how
personalization works.  Or in other words, it needs to implement a contract that indicates it can accept 
personalization data---but no duck typing!  In vanilla JS React, this contract is the propTypes, but in TypeScript 
we can accomplish much more.
  
The most important part in this code is the first type declaration.  I created a [type alias](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases)
called `HOCWrapped` because I intend to reuse this, and I don't really want to have to look a this ugly code more than once:
 
`type HOCWrapped React.ComponentClass<PWrapped & PHoc> | React.SFC<PWrapped & PHoc>;`

This type alias includes two different types, `React.ComponentClass` and `React.SFC`.  Those two types are separated
by a vertical bar, so taken together it is a [union type](https://www.typescriptlang.org/docs/handbook/advanced-types.html#union-types) 
meaning our `withPersonalization` HOC will work with *either* a [traditional React Component Class or a Stateless Functional Component](https://facebook.github.io/react/docs/components-and-props.html#functional-and-class-components).
   
The powerful part of this is the ampersand: `PWrapped & PHoc`.  `PWrapped` represents the *type of the props that the wrapped
component uses, and that we want to expose to the caller*.  `PHoc`, on the other hand, is the props type that the HOC wrapper will
implement, but will hide from the caller.  This type is an (intersection type)[https://www.typescriptlang.org/docs/handbook/advanced-types.html#intersection-types], 
meaning you pass as props an object that implements *both* the type `PWrapped` and the type `PHoc`.  

The angle brackets around the union type indicate that we're creating a generic type.  So `React.ComponentClass<PWrapped & PHoc>` 
ultimately means "A React Component that takes props which implements PWrapped and PHoc".  

This is indeed complex and confusing.  So let's convert this declaration to a paragraph in English:

<pre>
type HOCWrapped<PWrapped, PHoc> = React.ComponentClass<PWrapped & PHoc> | React.SFC<PWrapped & PHoc>;

export function withPersonalization<P, S>(Component: HOCWrapped<P, IWithPersonalizationProps>): React.ComponentClass<P> {
   ...
}
</pre>

> *withPersonalization* is a function that takes a component or a higher order function as an argument, and we'll 
call this argument the "wrapped component".  The wrapped component takes properties of type 
P-merged-together-with-IWithPersonalizationProps, and and uses a state of type S.  The output of this function 
is a component that accepts only P (and not IWithPersonalizationProps).

React wants us to pass most everything into a component via props, so that's why we're going through this
complex typing exercise.  We want to make sure we only accept props of type P from a caller, and reject
props that are of type IWithPersonalizationProps.  But we also want to make sure that the wrapped component
understands IWithPersonalizationProps.  

If you look closely, this wrapper accomplishes analogous to what [destructuring assignment] https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) syntax does
for variables—except that this operates on types.  We've created a type that combines `PWrapped` and a `PHoc` into
one props object, then we extracted the wrapped component's prop type AND the HOC prop type (using the ampersand).

So I said I wouldn't modify the `Welcome` component, but I guess that's not the case.  We need to make sure it 
implements `IWithPersonalizationProps`.  (Notice that I'm exporting the unwrapped component as well as the 
wrapped component as a default export.  That makes it easier to test later without elaborate mocking.)  Here's
the new, wrappable component:

<script src="https://gist.github.com/mikebridge/fa76d40ac7a0fc90530a669455d7eda5.js"></script>

Let's do a sanity check.  When we use this thing, does it accept `onClick`, but not `name`?  Here's what it looks
like in IntelliJ:  

<figure>
 	<img src="/images/typescript/typecheck1.png">
 	<figcaption>'name' is not supposed to be there: good!</figcaption>
</figure>

<figure>
 	<img src="/images/typescript/typecheck2.png">
 	<figcaption>Fixed before we even left the editor!</figcaption>
</figure>



