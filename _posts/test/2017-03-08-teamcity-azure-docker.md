---
layout: article
title: TeamCity Server on Azure & Docker, Part 1 
tags: [continuous integration, docker, teamcity, azure, containers, dotnet]
categories: articles
comments: true
excerpt: Set up Continuous Integration on a Azure ACS
image:
  teaser: intellij/react-native-400x250.png
ads: true
---

I want my dotnet app to be tested every time I push
a change to github.  And I want production-ready changes
to be deployed automatically to the production environment. There 
are many great SaaS services that do this for you, but you can 
create your own with TeamCity.

I haven't had much experience with Azure's conainer management
tools, so I though I'd try setting something up.

TeamCity, in general, has a central server that runs a 
site on TomCat.  You configure "builds" there and
tell the server how to run them.  Usually they are executed
on other VMs that the server spins up.

An easy way to set this up is to create a Virtual Machine
to run the TeamCity server, but we're going to try this
in docker.  The advantage of docker is the portability of
the software we run on it.  The thing that makes it portable
is going to be our use of Azure 
[https://github.com/Azure/azurefile-dockervolumedriver](File Storage)
driver.
==============

## Create an ACS Container


## Mount teh azure file system 
 docker volume create --name teamcity_volume -d azurefile -o share=tcshare

 Error looking up volume plugin azurefile: legacy plugin: plugin not found

This means

SEE: https://github.com/Azure/azurefile-dockervolumedriver/tree/master/contrib/init/systemd

wget https://github.com/Azure/azurefile-dockervolumedriver/releases/download/v0.5.1/azurefile-dockervolumedriver
wget https://raw.githubusercontent.com/Azure/azurefile-dockervolumedriver/master/contrib/init/systemd/azurefile-dockervolumedriver.service
wget https://raw.githubusercontent.com/Azure/azurefile-dockervolumedriver/master/contrib/init/systemd/azurefile-dockervolumedriver.default


==============


On Azure:

Create an Azure Container Service with a slow machine

On your machine

> docker pull jetbrains/teamcity-server
docker run -d --name teamcity -p 8111:8111 jetbrains/teamcity-server

Check that it works:

http://localhost:8111

Stop it 

Docker stop teamcity

Ok, now set up docker swarm:

Stop Docker locally (you’ll know it’s not working if you get “address already in use”.

ssh -i ~/.ssh/azure_rsa -fNL 2375:localhost:2375 -p 2200 ci@teamcitymgmt.eastus.cloudapp.azure.com


Create a disk on azure
...

Log in to the server to mount a disk
(see);
https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-linux-add-disk

Azure File Storage plugin, we can mount Azure File Storage shares as directories on your host’s file system and make it available to containers, which can now all make use of the Docker volume created through the plugin.

Fdisk …

$ sudo mkdir /teamcity
$ sudo mount /dev/sdc1 /teamcity
$ sudo -i blkid
$ sudo mkdir /teamcity/data
$ sudo mkdir /teamcity/log

Now deploy into the container, using the new drive

On client:
> SET DOCKER_HOST=:2375


docker run -d --name teamcity -p 8111:8111 jetbrains/teamcity-server
docker run -it --name teamcity   \
       -e TEAMCITY_SERVER_MEM_OPTS="-Xmx2g -XX:MaxPermSize=270m -XX:ReservedCodeCacheSize=350m" \
       -v /teamcity/data:/data/teamcity_server/datadir  \
       -v /teamcity/logs:/opt/teamcity/logs   \
       -p 80:8111 \
       jetbrains/teamcity-server

docker run -it --name teamcity -e TEAMCITY_SERVER_MEM_OPTS="-Xmx2g -XX:MaxPermSize=270m -XX:ReservedCodeCacheSize=350m" -v /teamcity/data:/data/teamcity_server/datadir -v /teamcity/logs:/opt/teamcity/logs -p 80:8111 jetbrains/teamcity-server


.. Need to make sure there’s a public ip address
