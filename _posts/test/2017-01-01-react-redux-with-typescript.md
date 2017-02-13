---
layout: article
title: Is it worthwhile to use TypeScript with React & Redux? 
tags: [typescript, javascript, react, redux, react-redux]
categories: articles
comments: true
excerpt: Is it worth it to use TypeScript with React & Redux
ads: true
---

Is it worthwhile to use TypeScript with React & Redux?  There
are many things to consider when choosing a development environment,
but two of the most important things to consider are how easy it
 is to write maintainable code and how fast you can write the code.

Although I think there's no question that a 
I prefer static types over dynamic types for a large project,
and I've been wondering if the investment is worth it.

There are really two big risks of adopting TypeScript in a React
environment.  Firstly, when you use a vanilla-JavaScript library,
you need a type definition file.  Are these widespread enough
so that you're not constantly struggling to write your own 
definition files to use an open source library.  You can't
really go half-way with TypeScript when it comes to importing
libraries.

Also, React-Redux is essentially a monkey-patching library.  
Is it wise to try to impose types on something like that?


Here are the things I've found to be useful:

1) If you aren't happy with how your current editor/IDE handles
typescript, you should try [IntelliJ](https://www.jetbrains.com/idea/).
IntelliJ makes full use of the static typing that TypeScript provides
to make autocompletion, refactoring and testing 

2) Use tslint while you're editing your code.  I turn some of the crazy
  options off like the OCD whitespace settings, but it points out enough
  obscure problems to make the extra effort worthwhile.  It's easy to 
  get lazy when TypeScript is essentially optional, so it's good to have
  the extra discipline....
  
Here's what gets you 80% of the way in React and Redux.
 
## React

React is a little more TypeScript-friendly than Redux.   



