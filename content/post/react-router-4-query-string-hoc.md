+++
title = "TypeScript and React, Part 5"
subtitle = "HOC Example: Access the Query String"
categories = ["react", "typescript"]
description = "A TypeScript Example of a Higher Order Control to access the Query String"
date = "2017-05-16"
aliases = ["articles/react-router-4-query-string-hoc"]
+++

>
> This is part of this series of blog posts introducing **TypeScript** and **React**. Here's [Part 1](/articles/getting-started-typescript-react/).
>

React-router 4 [removed the query string parsing feature](https://github.com/ReactTraining/react-router/issues/4410) that was present in 
past versions.  This is a good example of where a React higher order control would 
make a good substitute.  Instead of forcing Components to take dependencies on both the `location` and a
query-string parser, making things hard to test and refactor, it'd be much easier to 
create a wrapper that does the boring parsing work for you and DRYs out your code.
 
This follows [the TypeScript HOC template I described in the previous post](https://mikebridge.github.io/articles/getting-started-typescript-react-4/), 
where a component implements an interface describing the props it expects to be injected
(in this case, a "params" object of type `any`).

[query-string](https://github.com/sindresorhus/query-string) was the first query-string parser I came up with via
Google:
 
<pre>
> yarn add query-string
</pre>

Here's a quick example with `react-router-dom` 4.0.9:
 
<script src="https://gist.github.com/mikebridge/3b1746a3208e0eb6bc3e1a00275ad941.js"></script>

... and some tests:
 
<script src="https://gist.github.com/mikebridge/32c3b47f07d29f33b6dbda32e2796157.js"></script>

That one line in the test shows how to wrap a component---normally it would be the default export:

<pre>
export default withQueryString(MyComponent);
</pre>

### More Reading:

* <a href="/articles/getting-started-typescript-react">Part 1: Getting Started with TypeScript and React</a>
* <a href="/articles/getting-started-typescript-react-2">Part 2: Simple React Components in TypeScript</a>
* <a href="/articles/getting-started-typescript-react-3">Part 3: Stateless React Components in TypeScript</a>
* <a href="/articles/getting-started-typescript-react-4">Part 4: React's Higher Order Controls in TypeScript</a>
* Part 4a A HOC example 
* Coming soon: TypeScript with Redux; Testing & Continuous Deployment
