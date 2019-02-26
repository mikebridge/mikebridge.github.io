+++
title = "TypeScript and React, Part 3"
subtitle = "Stateless React Components in TypeScript"
categories = ["react", "typescript"]
description = "Examples of Stateless React Components in TypeScript"
date = "2017-05-01"
aliases = ["articles/getting-started-typescript-react-3"]
+++

>
> This is part of this series of blog posts introducing **TypeScript** and **React**. Here's [Part 1](/articles/getting-started-typescript-react/).
>


TypeScript's [inferred typing](https://www.typescriptlang.org/docs/handbook/type-inference.html) gives you
 some flexibility on how you declare ["Stateless Function Components"](https://facebook.github.io/react/blog/2015/09/10/react-v0.14-rc1.html#stateless-function-components).  Here
 are some examples.
 

**Simple:**

<script src="https://gist.github.com/mikebridge/771f9fa8dc08e78fa39fff3cf2d5aba0.js"></script>

If you are wondering what the extra curly braces are for in the parameters of the three examples,
 `myvar` and `onClick` are extracted from props via 
 [destructuring assignment](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).  This 
 next one is a little strange, because the types are declared inline.

**With inline declaration of props:**

<script src="https://gist.github.com/mikebridge/7c0fc67c6298704e2a8fc660d7afe98d.js"></script>

The thing that may be unclear about that syntax is that this types the `props` object, and therefore
only types the `myVar` and `onClick` variables *indirectly*.  Here's a 
[fuller explanation](https://blog.mariusschulz.com/2015/11/13/typing-destructured-object-parameters-in-typescript).  I think it's
clearer to use an interface instead:

**With an interface for props:**

<script src="https://gist.github.com/mikebridge/5f8ca49538df873da49c3205d421bc18.js"></script>

**Verbose:**

It's good to know what kind of types are being created, especially when combining these
with higher-order functions.  This is more explicit:

<script src="https://gist.github.com/mikebridge/7ce09ad441c2e239e6abd330c115561a.js"></script>

**Very Verbose**

This is a little unwieldy.  Probably I wouldn't ever explicitly declare that you can click 
on either an HTMLElement or a JSX.Element, or that I'm wrapping a div type:

<script src="https://gist.github.com/mikebridge/9c01b5d703a245c209a29701b7822610.js"></script>

**Tests**

For completeness, here are the tests:

<script src="https://gist.github.com/mikebridge/021db91e6524690512339c2c3e2df233.js"></script>

### More Reading

* <a href="/articles/getting-started-typescript-react">Part 1: Getting Started with TypeScript and React</a>
* <a href="/articles/getting-started-typescript-react-2">Part 2: Simple React Components in TypeScript</a>
* Part 3: Stateless React Components in TypeScript
* <a href="/articles/getting-started-typescript-react-4">Part 4: React's Higher Order Controls in TypeScript</a>
* Coming soon: TypeScript with Redux; Testing & Continuous Deployment

