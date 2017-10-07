---
layout: article
title: Configure Azure for Deployment from TeamCity
tags: [azure teamcity continuous-deployment dotnet core slots]
categories: articles
comments: true
excerpt: Set up Azure To Deploy from TeamCity
image:
  teaser: teamcity/wormhole-400x250.png
ads: true
---

*October 7, 2017*

>
> This is part of a series of blog posts on DevOps for **DotNet Core**, **TypeScript** and **React**.
>

I'm running a Continuous Deployment pipeline where checkins to a "production" branch on github 
automatically trigger deployments on Azure.  There are many moving parts to this, and this 
particular post describes just one part---how to copy a node and dotnet core site to Azure.  

TeamCity interacts with Azure in three ways:

1) Copy the deployment to a "slot" before it goes live
2) Warm up the web site that we deployed on the slot
3) Run the database migration
2) Swap the slots and make the web site live 

## Copy to a Slot

A "slot" can be used like a separate non-production web site that you can copy files to, then
"warm up", then swap with the production site when you're satisfied that the update went successful.
I use SFTP to copy the whole site up, though you could easily use some other method.

You first need to create a slot.

... describe setup

## Warm up the web site

... via curl

## Run the database migration

##






I will assume you already have an Azure setup, and a 

Coming up in this series: *Set up TeamCity for DotNet Core*; *Continuous Deployment of DotNet Core 
to Azure via GitHub*.



