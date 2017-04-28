---
layout: article
title: TypeScript and React, Part 1
tags: [react redux typescript]
categories: articles
comments: true
excerpt: Getting Started with TypeScript and React, Part 1
image:
  teaser: typescript/ts-400x250.png
ads: true
---

TypeScript with React: is it worth it?  I started using them together a few 
months ago with some hesitation.  I wanted the long-term benefits that strong typing 
brings to a larger code-base, but I always hesitate before wandering off of the main 
path.  When it comes to mainstream software development, it's usually best to stick 
with the group rather than struggling to survive in the woods on your own.  So, on 
balance, do you gain more than you lose?  

> **TL;DR**
> 
> Although I've had a few problems, the experience has positive overall.
>
> React + TypeScript work well together, but:
> 
> - Start out with [the TypeScript fork of create-react-app](https://github.com/wmonk/create-react-app-typescript) 
> - Use an editor that does TypeScript code analysis well, like [IntelliJ Ultimate](https://www.jetbrains.com/idea/) 
> - Use [TSLint](https://palantir.github.io/tslint/) religiously.
> - Get a good understanding of TypeScript's [basic](https://www.typescriptlang.org/docs/handbook/basic-types.html) and
[advanced](https://www.typescriptlang.org/docs/handbook/advanced-types.html) types.  You'll need most of them for React + Redux

I identified several risks with switching to TypeScript:

## 1) Are the development tools be hard to set up and maintain?  

This was probably the biggest question for me.  I have no desire to waste several days in 
webpack build-configuration hell, then more days in editor-configuration hell, test-configuration hell,
 continuous integration hell and deployment hell—and then to repeat the whole process every
 few months when all the tools get updated.

### create-react-app-typescript 
 
Facebook heard from developers that they found it difficult to start working with React because
it is complex to create a decent development environment.  So they created [create-react-app](https://github.com/facebookincubator/create-react-app) 
as a starting point.  This gives developers the ability to run a web app, package it, test it, etc. with no configuration, 
and it's maintained by Facebook, so you can future-proof your environment by allowing them to keep it up-to-date for you. 

Fortunately Will Monk has [created a fork specifically designed to work with TypeScript](https://github.com/wmonk/create-react-app-typescript),
which so far has been well maintained.  There's a bit of risk there because he's just one
person, but it seems to have some decent adoption so I hope it will continue
to be a going concern.  

So to get started with TypeScript and React, this is all you need to do:

<pre>
    > npm install -g create-react-app

    > create-react-app my-app --scripts-version=react-scripts-ts
    
    > cd my-app/
    
    > npm start
</pre>

Now you have a working React/TypeScript web site.  And the build process is taken care of:

<pre>
    # start the server on port 4000, with hot-reloading enabled
    > npm start

    # run jest tests continuously
    > npm test

    # package a build for deployment
    > npm run build

    # heck, generate the test coverage report
    > npm test -- --coverage
</pre>


... plus [lots more](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#updating-to-new-releases).
This works for me on both Linux and Windows. With `create-react-app` you can delegate the 
unpleasant task of build-script maintenance to Facebook, but if you should decide some day 
that you need some special configuration on your own, you can take over the configuration 
by "[ejecting](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#npm-run-eject)"
and maintaining the scripts yourself.
 
### IntelliJ / WebStorm

I can't say enough good things about the [JetBrains](https://www.jetbrains.com/) family of 
editors.  I've used most of the mainstream editors over the years and [IntelliJ Ultimate](https://www.jetbrains.com/idea/) is 
the most powerful—certainly for editing TypeScript.  It [TSLint](https://palantir.github.io/tslint/)s
your code in real time, gives you coding suggestions, provides *real* autocompletion/intellisense (not just suggestions 
from a dictionary), and automates code refactoring (decently, though not perfectly), among
many other things.  If you don't use an editor that takes full advantage of the kinds of code analysis that TypeScript makes available, you're leaving
money on the table.  Getting real-time feedback on your code as you type is invaluable. 

I don't actually compile TypeScript from within IntelliJ, though that's an option.  Instead I let
`create-react-app` do all the heavy lifting.  Usually run `npm test` in one terminal to
run tests continuously, and `npm start` in another to update my browser whenever code 
changes.  
 
[WebStorm](https://www.jetbrains.com/webstorm/) is a subset of IntelliJ optimized for 
JavaScript.  IntelliJ supports almost every major language, but if you don't do anything
but JavaScript you could just stick with WebStorm.

The one downside is that I have not been able to figure out how to run TypeScript jest
tests in debugging mode from within IntelliJ.  (I can, 
however, [debug code via the web server](https://blog.jetbrains.com/webstorm/2017/01/debugging-react-apps/)) 

## 2) Will there be gnashing of teeth because a JS library has no TypeScript type definition files? 
 
 You need type definition files in TypeScript to use plain javascript libraries.  These work
 like header files in languages like `C`, and they are maintained in a [huge github
 repository](https://github.com/DefinitelyTyped/DefinitelyTyped/), often by third-parties.  With 
 [TypeScript 2](https://blogs.msdn.microsoft.com/typescript/2016/06/15/the-future-of-declaration-files/) 
 these are now trivial to acquire with `npm` or `yarn`, and often you don't even realize the typing
 files are there—they just show up with the main package.
 
 I only have anecdotal evidence, but I have not had many problems with missing type 
 definitions so far.  In one case (with [auth0-js](https://www.npmjs.com/package/@types/auth0-js) I've downloaded an outdated `.d.ts` file someone wrote 
 and modified it myself.  I also had difficulty with 
 adding `react-intl` to the tool chain, but I eventually [came up with a solution](https://mikebridge.github.io/articles/typescript-i18n-react-intl/).  
 
 I've also rewritten a few open-source JavaScript excerpts into TypeScript rather than write a type definition file for them.  Since
 TypeScript is a superset of JavaScript, this has always been an easy task.
 
 For the most part, the React-Redux environment's types have been fairly well maintained and
 done in a very sophisticated manner.  The 
 issue is that the types that are superimposed on them can be complex.  Figuring out how
 these advanced types interact has been the most time consuming for me, but I'll provide more 
 information on how this works in react & redux in future blog posts.

## 3) Is it a headache to impose types on third-parties' untyped npm hairballs? 
   
Everyone's pulled in nightmare JavaScript code from npm that accepts just about anything and 
can return who-knows-what.  TypeScript has two things going for it---a very powerful type system
and an escape hatch.

The TypeScript type system is a brilliant piece of work, but it requires some investment to understand 
how it works.  A good starting point is [Basarat Ali Syed's online book](https://basarat.gitbooks.io/typescript/)
I've found that in most cases, union types, intersection types and generic types will make
sense out of most [hairballs](https://www.youtube.com/watch?v=rI8tNMsozo0), but sometimes these can get complex, 
especially when wrapping Components in HOCs or using Redux.  I'll provide some example in later blog posts on how 
this can work smoothly.  
 
Fortunately, there's always an escape hatch: [`any`](https://www.typescriptlang.org/docs/handbook/basic-types.html#any).
If you can't figure a type out, you can always cast an object to any:

<pre>
    const monkeyDoo = (someObj as any).monkeyPatchedFunction();
</pre>

I admit that I've encountered situations 
where I've been flummoxed by nested type definitions or protean return objects and I've 
taken the easy road out by casting to `any`.  And I've found that some libraries are quite 
happy to pollute the props of wrapped components---which isn't a big deal if you're working
in plain JavaScript, but a real source of complexity in TypeScript.

*Coming soon: Creating simple React components with TypeScript.*
