---
layout: article
title: Throttling in ReactJS with RxJS
tags: [react rxjs typescript observable]
categories: articles
comments: true
excerpt: Two ways to throttle events in ReactJS 
image:
  teaser: typescript/redux-observable-400x250.png
ads: true
---

*June 26, 2017*

I have been using [redux-observable](https://redux-observable.js.org/) as middleware to process Redux actions
such as requests for ajax calls, and I wanted to start applying the underlying Rx library, RxJS, to other
problems, like throttling `mouseover` events.

The advice is to [avoid using subjects](https://medium.com/@benlesh/on-the-subject-of-subjects-in-rxjs-2b08b7198b93) wherever
possible.  A canonical work on that is [here](http://davesexton.com/blog/post/To-Use-Subject-Or-Not-To-Use-Subject.aspx).

If we're using a simple html tag such as `div`, this is easy to implement with one of the set of sanctioned `Rx.Observable` creator
functions---in this case, we use an old-fashioned [ref](https://facebook.github.io/react/docs/refs-and-the-dom.html) to 
get a reference to the `div` element, then we pass it to `Observable.fromEvent`:

<script src="https://gist.github.com/mikebridge/8727ecf8f32b9dde63055a52b8c2c85e.js"></script>

There's a simple working version of this on [codepen](https://codepen.io/mikebridge/pen/XgeMwK/).

But many components ask for a function which will be called when an event, such as a click, occurs.  In this case
it seems like the most sensible thing to do is to use a `subject`:






