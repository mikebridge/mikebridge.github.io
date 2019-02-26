+++
title = "TypeScript and React, Part 4"
subtitle = "React Higher Order Components in TypeScript"
categories = ["react", "typescript"]
description = "React Higher Order Controls in TypeScript"
date = "2017-05-04"
aliases = ["articles/getting-started-typescript-react-4"]
+++

>
> This is part of this series of blog posts introducing **TypeScript** and **React**. Here's [Part 1](/articles/getting-started-typescript-react/).
>

>
> *Edit, Feb 26, 2019*: Nobody on the team liked this approach as much as [Render Props](https://reactjs.org/docs/render-props.html), so we've switched to that.
> Our experience was that HOCs can create mind-bending types.
>

React's [Higher Order Component](https://facebook.github.io/react/docs/higher-order-components.html) pattern
is a technique to help developers design better components and component interactions.  Refactoring out cross-cutting 
concerns from a complex component with HOCs touches on at least three or four of 
the [SOLID principles](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)), but creating a cohesive
design can be difficult without proper typing.  And it's can be especially difficult in React when using 
plain, old untyped JavaScript because most of the data that goes into a component gets mashed into a single 
untyped Props object.  The more interaction there is, the harder it gets to keep track of, especially as
the code changes over time.  Fortunately, TypeScript provides some tools and some structure that can help keep 
our component design and architecture clean as our software evolves and changes. 

Say, for example, that you want to create a component that displays the message "Welcome {username}" and sends the 
user to a "User Settings" page when that message is clicked.

The first iteration might be to write a simple component that displays the logged-in user's information:

<script src="https://gist.github.com/mikebridge/e6dc04180f9a1cd42410d961d1971ccf.js"></script>

And here are some tests:

<script src="https://gist.github.com/mikebridge/b9bb34e9a2f60c871ab7e38e812c7c12.js"></script>

That's straightforward enough.   But there's still some logic to implement:

1. If the user is not logged in, we want to hide this component.
2. If the user *is* logged in, we need to access his or her name, e.g. from Redux.
2. We need to trigger navigation when the component is clicked, e.g. via [react-router](https://github.com/ReactTraining/react-router).

Where should all that functionality go?  It would be [easy](https://www.youtube.com/watch?v=rI8tNMsozo0) to spew all 
this logic right into the Welcome component by [`connect`-ing](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) it to
our Redux store and then also referencing the router directly.  However, there are a few good reasons that this might not be a 
 good idea:

- We'll likely need to personalize other components later, so this is probably a cross-cutting concern
- The `Welcome` component is a **[dumb / presentational](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)**
component, and this extra logic probably belongs in a separate **smart / container** component.  Our simple component shouldn't
know about datastores or site navigation---it should just display data.  

In terms of SOLID principles, we want to avoid adding this logic to our `Welcome` component because a) it would 
compromise its *Single Responsibility*—it's just supposed to display a personalized link—and b) it 
should be *Closed to Modification but Open to Extension*—we don't want to dig around in this component every 
time we need to add a new feature.

Let's say our software already puts the user information in the Redux store under the `user` key.  A HOC wrapper might
grab that object if it exists and inject it (hey, there's the [D](https://en.wikipedia.org/wiki/Dependency_inversion_principle) from SOLID too!) 
into the subcomponent.  I would describe this functionality as "personalization", so I'll call this HOC 
`withPersonalization`.  Here's a first step, where the personalization is hardcoded (later we'll introduce Redux):

<script src="https://gist.github.com/mikebridge/93dbc41f1afc2dd4ce64f08106eefbab.js"></script>

There are some type declarations there that hurt the eyes (*Disclaimer: if you're a Rails dev they may 
land you in the hospital*), so let's remember what we're trying to accomplish with this first iteration:

- The goal is to provide an interface to props that a developer can see, but also have another interface
to props that is only used internally (e.g. by redux).  So when a developer includes our `<Welcome />` 
component, we would like to make it clear to him that the `onClick` function is available (e.g via autosuggest),
but we also need to ensure that the the `name` is hidden from him, but visible to redux's `connect`.
  
- When we wrap a component in `withPersonalization`, we want to ensure that the underlying component is aware of how
personalization works.  Or in other words, it needs to implement a contract that indicates it can accept 
personalization data---no lazy duck typing!  In vanilla JS React, this contract is sort-of provided by propTypes, but 
in TypeScript we can accomplish a lot more.
  
The most important part in this code is the first type declaration.  I created 
a [type alias](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases) called `HOCWrapped` because I 
intend to reuse this, and I don't really want to have to look a this ugly code more than once:
 
`type HOCWrapped React.ComponentClass<PWrapped & PHoc> | React.SFC<PWrapped & PHoc>;`

This type alias includes two different types, `React.ComponentClass` and `React.SFC`.  Those two types are separated
by a vertical bar, so taken together it is a [union type](https://www.typescriptlang.org/docs/handbook/advanced-types.html#union-types) 
meaning our `withPersonalization` HOC will work with *either* a [traditional React Component Class *or* a Stateless Functional Component](https://facebook.github.io/react/docs/components-and-props.html#functional-and-class-components).
   
The powerful part of this is the ampersand: `PWrapped & PHoc`.  `PWrapped` represents the *type of the props that the wrapped
component uses, and that we ultimately want to expose to the caller*.  `PHoc`, on the other hand, is *the props type that the HOC wrapper will
implement, but will hide from the caller*.  The two types, joined together with an ampersand, is an [intersection type](https://www.typescriptlang.org/docs/handbook/advanced-types.html#intersection-types), 
meaning you pass as props an object that implements *both* the type `PWrapped` and the type `PHoc`.  

The angle brackets around the union type indicate that we're creating a generic type.  So `React.ComponentClass<PWrapped & PHoc>` 
means "A React Component that takes props which implements PWrapped and PHoc".  

This is indeed complex and confusing, especially if you haven't worked in typed languages like C#, Java or Scala.  So 
let's convert this declaration...

<pre>
    type HOCWrapped&lt;PWrapped, PHoc> = React.ComponentClass&lt;PWrapped & PHoc> | React.SFC&lt;PWrapped & PHoc>;

    export function withPersonalization&lt;P, S>(
           Component: HOCWrapped&lt;P, IWithPersonalizationProps>): React.ComponentClass&lt;P> {
       ...
    }
</pre>

... into a paragraph in English:

> *withPersonalization* is a function that takes a component class or a higher order function as an argument, 
> and we'll call this argument the "wrapped component".  The wrapped component takes properties of type 
> P-merged-together-with-IWithPersonalizationProps (i.e. a props object that's passed in must implement both 
> types), and the wrapped component uses a state of type S.  The output of this function is another 
> component class that accepts only objects of type P as props (and does not accept props with attributes 
> appearing in IWithPersonalizationProps).

React wants us to stuff most everything into a component via props, so that's why we're going through this
complex typing exercise.  We want to make sure we only accept props of type `P` from a caller, and reject
props that are of type `IWithPersonalizationProps`.  But we also want to make sure that the wrapped component
understands `IWithPersonalizationProps`.

This wrapper accomplishes something analogous to what [destructuring assignment](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) syntax does
for variables—except that it operates on types.  We've created a type that merges `PWrapped` and a `PHoc` into
one props type, then we extracted the wrapped component's prop type AND the HOC prop type back into their
original types again. Gimme an [I](https://en.wikipedia.org/wiki/Interface_segregation_principle)!

So I said I wouldn't modify the `Welcome` component, but I guess that's not the case.  We won't change functionality
though---just solidify its communication with other components.  We need to make sure it 
is expecting the personalization, so it should know about `IWithPersonalizationProps`.  (Notice that I'm exporting the 
unwrapped component as well as the wrapped component as a default export, so it's easier to test without elaborate 
mocking.)  Here's the second iteration of our component:

<script src="https://gist.github.com/mikebridge/fa76d40ac7a0fc90530a669455d7eda5.js"></script>

Let's do a sanity check.  When we use this thing, does it accept `onClick`, but not `name`?  Here's what it looks
like in IntelliJ:  

{{<figure src="/images/typescript/typecheck1.png" caption="'name' is not supposed to be there: check!" caption-effect="fade" caption-position="bottom">}}

{{<figure src="/images/typescript/typecheck2.png" caption="Fixed before we even left the editor!" caption-effect="fade" caption-position="bottom">}}

>
> TypeScript makes `propTypes` obsolete.
> 

So let's take the next step---let's take the user data from Redux:
 
<script src="https://gist.github.com/mikebridge/9abd8407a6f022c8f9d213d15c1c0426.js"></script>

In order for Redux to type the returned connected component correctly, we need to explicitly declare the ownProps, 
even though we don't use it.  This allows TypeScript to infer the types correctly in `connect` without requiring 
explicit definitions.  ("ownProps" seems to be the standard term distinguishing the props that are
defined locally by a component from props that are defined elsewhere, like redux.)

<pre>
const mapStateToProps = (state: any, ownProps: P): IWithPersonalizationProps => ({
     name: state.user.name
});
</pre>

Here are my tests:

<script src="https://gist.github.com/mikebridge/3bb614064da11f1c85f4a13dc09105d0.js"></script>

All tests pass: great!

There's still one more thing to do, which is connect to the router, so that we satisfy our last use-case.  This 
is essentially the same strategy as our `withPersonalization` HOC, but we'll also pass a second `routeWhenClicked`
property:

<script src="https://gist.github.com/mikebridge/96a4487aa185343c13bfba5fcc3b431b.js"></script>


Here's the final version of the welcome control, with the cross-cutting concerns factored out into HOCs:

<script src="https://gist.github.com/mikebridge/0c30a19f8c6653c07349e901e97689bc.js"></script>

The last line is where all the pieces come together.  You can pull in the new `<Welcome />` component like this:

<pre>
   import Welcome from "./welcome";
</pre>


*Side note: I moved the intersection type into a type alias.  However, there's one oddity---type destructuring didn't work 
correctly when I stacked the HOCs together unless I included an empty "OwnProps" declaration. That's where 
the `interface IWelcomeOwnProps {}` declaration comes from.*

* Edit: [Here's an example](/react-router-4-query-string-hoc) of replacing react-router's missing QueryString
parsing using an HOC.

### Conclusion

Placing functionality into separate HOCs makes it much easier to reuse React components.  Here we have
two reusable HOCs and a very simple base "Welcome" component, all of which are simple to comprehend
and modify.  TypeScript's compile-time (and edit-time) checks helped us separate out the concerns 
confidently, with the knowledge that it will be much easier to modify clean code like this in the future.  And
TypeScript didn't clutter up our code, either---many of the types were inferred rather than explicitly declared.

### More Reading:

* <a href="/articles/getting-started-typescript-react">Part 1: Getting Started with TypeScript and React</a>
* <a href="/articles/getting-started-typescript-react-2">Part 2: Simple React Components in TypeScript</a>
* <a href="/articles/getting-started-typescript-react-3">Part 3: Stateless React Components in TypeScript</a>
* Part 4: React's Higher Order Controls in TypeScript
* Coming soon: TypeScript with Redux; Testing & Continuous Deployment
