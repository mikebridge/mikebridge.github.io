---
layout: article
title: DotNet Core & TeamCity
tags: [intellij react-native windows]
categories: articles
comments: true
excerpt: Continuous Integration with TeamCity
image:
  teaser: react-intl/globe-400x250.png
ads: true
---

I've run TeamCity for years on AWS, and now that I'm setting
up some new projects on Azure, I wanted to modernize the deployment
a bit.  However, after setting the whole thing up on Azure, I realized that thewhole thing was going to be pretty expensive.  Fortunately, I have some
hardware lying around the office that would work almost as well.

Here's the boring part.  In order to get TeamCity to be able to hear
from Github, you need to have it accessible via the web.  If you're behind
a firewall, this is slightly more work, but much nicer.

Normally I proxy stuff through the super-cheap droplets from Digital Ocean---
it's like $5 a month---but in this case I'm using an Ubuntu VM on Azure, so
I'll set this up there.

Here's how this is going to look when it's done:

Github

Azure (or AWS or Digital Ocean)

Firewall

Local Network
  - Docker Host 
  - Container in the Host running TeamCity Server and Agent
