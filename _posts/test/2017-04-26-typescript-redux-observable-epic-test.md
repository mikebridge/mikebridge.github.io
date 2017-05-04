---
layout: article
title: Testing Redux-Observable Epics
tags: [react typescript redux-observable epic test]
categories: articles
comments: true
excerpt: Creating Jest tests for Redux-ObservableAjax Epics
image:
  teaser: typescript/redux-observable-400x250.png
ads: true
---

*Apr 26, 2017*

Here's a way to test a [redux-observable](https://redux-observable.js.org/) epic that 
performs an ajax call.

This simplified TypeScript
example has three actions: `USER_LOAD_REQUEST` is the result of a request to load a user, 
`USER_LOAD_RESULT` indicates a successful response, and `USER_LOAD_ERROR` holds an 
ajax error.

<script src="https://gist.github.com/mikebridge/7509ca969e4e33f18149432257fbe8fd.js"></script>

Testing is straightforward with ReduxObservable's 
[dependency injection](https://redux-observable.js.org/docs/recipes/InjectingDependenciesIntoEpics.html) 
parameter.  In this case, I'm injecting the `ajax.getJSON` property into the epic.

Mocking the AJAX call involves either wrapping a plain JS object or a plain JS `Error` in an 
`Observable`.

<script src="https://gist.github.com/mikebridge/b27933835ddb87b9d859e37419b36801.js"></script>

Note the [`done`](https://facebook.github.io/jest/docs/asynchronous.html) parameter---this is a way of 
signaling to jest that an async call has completed.



