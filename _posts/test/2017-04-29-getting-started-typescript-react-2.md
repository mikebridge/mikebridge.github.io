---
layout: article
title: TypeScript and React, Part 2
tags: [react redux typescript]
categories: articles
comments: true
excerpt: Simple React Components in TypeScript
image:
  teaser: typescript/ts-400x250.png
ads: true
---

## A Simple React Component in TypeScript

*Apr 29, 2017*

* Part One of this series of blog posts introducing **TypeScript** and **React** is [here](/articles/getting-started-typescript-react/).

It's not too hard (any more) to get started with TypeScript and React.  This post assumes 
that you know a little about JavaScript and a little about React—if you haven't created 
an app already you can do it with [create-react-app-typescript](`https://github.com/wmonk/create-react-app-typescript`):

<pre>
    > npm install -g create-react-app
    > create-react-app my-app --scripts-version=react-scripts-ts    
    > cd my-app    
    > npm start    
</pre>

## The Component

Here's an example component called, well, `exampleComponent.tsx`: 
  
<script src="https://gist.github.com/mikebridge/b1d4f195dfa7b6fc8f0ae31682c8fcf8.js"></script>  

You'll notice that this is almost the same as it would be in vanilla JavaScript, except
for those type definitions.

Include it in `App.tsx` with this code:

<pre>
   import ExampleComponent from "./example/exampleComponent"; // use the "default" export
   // OR: import {ExampleComponent} from "./example/exampleComponent"; // use the explicit export
   // ...
   
   <ExampleComponent onCounterIncrease={(count) => console.log(count)} />
</pre>

If you open the console and click on the button, you should see the counter increase.

**Typing Props and State**  

When you create a `Component` class, you should create types for the props and the
state.  Here we're requiring that the caller provide the function `onCounterIncrease`, which takes the
current count number and returns nothing.  The `text` variable is a string, and it's
optional (hence the "?" in the type).

You can define a type as a TypeScript [`interface`](https://www.typescriptlang.org/docs/handbook/interfaces.html):

<pre>
    interface IMyProps {
        text?: string; // an optional text field
        myFunc: (count: number) => void; // a required function
        myObj: ISomeClass; // an object whose type is defined by the inteface ISomeClass
    }
</pre>
 
The nice thing about the [function declaration](https://www.typescriptlang.org/docs/handbook/functions.html#function-types) for `myFunc` 
is that the caller has to match the signature of the definition. Passing zero parameters to `myFunc` will result in a compile-time 
error.  If you decide you don't care about the function signature, you could also
declare `myFunc` as a `Function`.  Or even `any`, if you were really lazy.

> Although Bob Martin is usually a [great](https://www.amazon.ca/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) source of advice on most things coding-related,
> he [doesn't like "I's" in interfaces](http://stackoverflow.com/questions/5816951/prefixing-interfaces-with-i#answer-5817904).  Nor
> does he like [interfaces in general](http://blog.cleancoder.com/uncle-bob/2015/01/08/InterfaceConsideredHarmful.html).  I
> happen to like both.

React uses [a generic type](https://www.typescriptlang.org/docs/handbook/generics.html) to
define a `ReactComponent` class which uses the props and static types that you create.  If you haven't
used generic types before, they are a way of reusing one type definition with different 
subtypes. For example, an `Array` type might be able to hold any kind of object, but 
an `Array<string>` can only hold strings—and nothing else.  A React Component 
that's defined like the following can only accept props that conform to `IMyProps` and use a state that 
conforms to `IMyState`:

<pre>
    import * as React from "react";
    
    class MyComponent extends React.Component<IMyProps, IMyState> {
        // ...
    }    
</pre>
        
Sometimes you'll see definitions like **`class MyComponent extends React.Component<{}, {}>`**. This
means that this component doesn't use props or state—they must be empty.  **`class MyComponent extends React.Component<any, any>`** says
that you can pass anything for props and state and it will do no type checking.
                
**Static, Private, Public**

In TypeScript, `propTypes`, `contextTypes`, and `defaultProps` are a [static properties](https://www.typescriptlang.org/docs/handbook/classes.html#static-properties),
meaning they're created within the class definition, instead of on the prototype as in ES2015.

JavaScript still requires [contortions](http://javascript.crockford.com/private.html) to define
private and public members.  Fortunately these are [first-class parts of TypeScript](https://www.typescriptlang.org/docs/handbook/classes.html#public-private-and-protected-modifiers).  In
  this example, `onClick()` is invisible outside the component.

**Type Inference**

It sounds like the word "Type" in TypeScript refers to the amount of extra typing to get all those
 type definitions in place.  Fortunately, TypeScript is smart enough to [infer](https://www.typescriptlang.org/docs/handbook/type-inference.html) a lot
 of the types so that you don't have to explicitly provide them all the time.  This is why when you pass
 in the console logging function, you can type this:
 
<pre>
    (count) => console.log(count)
</pre>
     
...instead of this:

<pre>
    (count: number) => console.log(count)
</pre>

## Testing

I use [airbnb's enzyme](https://github.com/airbnb/enzyme) for testing react components:

<pre>
    > yarn add enzyme --dev 
    # OR
    > npm install enzyme --save-dev
</pre>

Here are a couple tests.  Note that nothing here is really TypeScript-specific.  You can run them by typing **`yarn test`** (or **`npm test`**). 

<script src="https://gist.github.com/mikebridge/ae7dafd955f2f3e206d9648a99649c82.js"></script>

Here's the thing: notice what happens if we forget to include our callback function—[IntelliJ](https://www.jetbrains.com/idea/) gives us a warning that
it's missing:

<figure>
 	<img src="/images/typescript/error.png">
 	<figcaption>'onCounterIncrease' is missing.</figcaption>
</figure>

TypeScript makes [propTypes unnecessary](https://facebook.github.io/react/docs/typechecking-with-proptypes.html), because
it will tell you at compile time—or *while you're typing it*, if you're using a TypeScript-aware editor—that you're missing something.  Or if you pass a function with the wrong signature (e.g.
with missing parameters, or parameters of the wrong type), it will tell you that too.

### More Reading

* <a href="/articles/getting-started-typescript-react">Part 1: Getting Started with TypeScript and React</a>
* Part 2: Simple React Components in TypeScript
* <a href="/articles/getting-started-typescript-react-3">Part 3: Stateless React Components in TypeScript</a>
* Coming soon: TypeScript with Redux, HOCs, Testing & Continuous Deployment with React & TypeScript*

