---
layout: article
title: Stateless React Components in TypeScript
tags: [react typescript]
categories: articles
comments: true
excerpt: Examples of Stateless Components in TypeScript
image:
  teaser: typescript/ts-400x250.png
ads: true
---

I like using TypeScript with React, but sometimes it's confusing, especially
when using the more advanced types.  For my own reference, here's an array of ways 
I came up with to declare **Stateless React Components** in TypeScript, in varying levels 
of verbosity.

Suggestions welcome!

**Simple:**

<script src="https://gist.github.com/mikebridge/771f9fa8dc08e78fa39fff3cf2d5aba0.js"></script>

**With inline declaration of props:**

<script src="https://gist.github.com/mikebridge/7c0fc67c6298704e2a8fc660d7afe98d.js"></script>

**With an interface for props:**

<script src="https://gist.github.com/mikebridge/5f8ca49538df873da49c3205d421bc18.js"></script>

**Verbose:**

<script src="https://gist.github.com/mikebridge/7ce09ad441c2e239e6abd330c115561a.js"></script>

**Crazy & Impractical**

<script src="https://gist.github.com/mikebridge/9c01b5d703a245c209a29701b7822610.js"></script>
