---
layout: article
title: An Event Aggregator in C#
tags: [c# ddd]
categories: articles
comments: true
excerpt: An Aggregator for DDD Domain Events in C#  
image:
  teaser: rx/reactive-extensions-400x250.png
ads: true
---

I have a web project that is made up of several modules, each of which generates DDD 
[domain events](http://codebetter.com/gregyoung/2010/04/11/what-is-a-domain-event/).  Right now
my C# project is small and divided into well-defined modules which run in a single process, so
using a message queue like Kafka is a bit of overkill.  But as the program scales, I may break
these modules out and run them as full microservices in their own processes.  At that point, I will 
need to use a real messaging system like Kafka or the Azure Event Hub. 

In the meantime, I need a simple Event Hub/Event Aggregator/Messaging Hub that can be easily replaced by
something else later on.  I found several libraries that were overly complicated or overly-opinionated, 
but ultimately came across this super-simple Reactive-Extension code 
from [Reactive.EventAggregator](https://github.com/shiftkey/Reactive.EventAggregator/blob/master/src/Reactive.EventAggregator/EventAggregator.cs).  I
modified the subscription slightly so that a caller doesn't need to know about Rx.

*The Simplest Thing that Could Possibly Work to handle Domain Events:*

<script src="https://gist.github.com/mikebridge/f6799ebed20160f72a3daf62f584d2ff.js"></script>

It maintains one aggregated stream of events as a `Subject` and makes use of [OfType](http://www.introtorx.com/content/v1.0.10621.0/08_Transformation.html#CastAndOfType) to 
divide it by type.

The tests show how to use it---multiple subscribers can listen for the same event, and you can
unsubscribe from events by calling `Dispose` (e.g. via `using`):

<script src="https://gist.github.com/mikebridge/a468735c966ea062487199cded301d1c.js"></script>

To make things easy, I wrapped the EventAggregator in a static class that can be accessed
from anywhere rather than using Dependency Injection, similar 
to [Udi Dahan's solution](http://udidahan.com/2009/06/14/domain-events-salvation/).

