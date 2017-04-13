---
layout: article
title: Continuous Deployment for DotNet Core with TeamCity 
subtitle: Set up TeamCity
tags: [continuous integration, docker, teamcity, azure, containers, dotnet]
categories: articles
comments: true
excerpt: Set up CD for DotNet Core, React, TeamCity and Azure
image:
  teaser: intellij/react-native-400x250.png
ads: true
---

> This is a multi-part blog post describing how to Continuous Deployment
> for a DotNet Core/React Single Page App using TeamCity, deployed on Azure, and
> using both Windows & Linux.

## The Goal

We use our own variant of [gitflow](http://nvie.com/posts/a-successful-git-branching-model/), 
where we have a central "master" branch that always has a working copy of of the application, and
from which we create small feature branches.  We will merge from "master" to "production" any
time we want to do a deployment.  

So the goals are:
 
1. Any time a modification is pushed to any branch other than `production`, the test will run and notifications will be sent.
2. Any time a modification is pushed to a the `production` branch, the web application will be deployed to Azure (and the 
database migrations will be run).

## Prerequisites

To get this running you need:
 
 - A free Linux machine or VM running Docker, and some basic Linux skills.
 - An Azure account with an App Service 
 - A React Single Page Application running on Node
 - A C# WebApi App  
 - SQL Server
 - A GitHub project containing the source.
 - Visual Studio 2015 or 2017.
 - DotNet tools
 - Node

There's some example code on github... (todo)

## Part 1: Set up TeamCity

Here is the way I want this to work:

- Every checkin to github will run all the C# Unit tests, C# Integration tests and JavaScript tests 
and notify the team of the build status.
- Every time the production branch is updated, a deployment to Azure is triggered.  This will
update the C# Api, the Node front end code, and migrate the databases.

So this will run tests:

```
$ git push origin master
```

And this will deploy the entire site:

```
$ git merge master production
$ git push origin production
```

Currently, the easiest way to set up TeamCity is via Docker.  I originally set up docker on Azure 
Container Services, but I could see that was going to be quite a bit more expensive than running
it on unused hardware in the office.   So now the whole thing runs on an old Dell machine
under my desk.  Although I'm running Linux containers on a Linux host, most of this will work 
on Windows containers too without too much modification.  Scripting in Linux is much more 
mature than on Windows.

So here's a demo project that we'll use to get this running:

__put github proejct here__

This is a very minimal project I created using [https://github.com/wmonk/create-react-app-typescript]
(the TypeScript fork of Facebook's create-react-app) and a very basic WebApi project.  The goal
is to deploy the React front-end on Node in Azure, the C# backend on a separate VM in Azure,
and using Azure SQL.

TeamCity has a server and a set of agents.  There are many ways to set up the agents, but 
in our simple case we'll run one server container, one agent container and one MSSQL container
on our docker host.

The server needs two things: a data directory, and an external database.  
I provisioned a SQL database on Azure and pointed to it, and I created the directory $HOME/data 
on the host.  I'm using [docker-compose](https://docs.docker.com/compose/), and the server setup 
is simple enough that it doesn't require a separate Dockerfile.  It 





Your machine
$ scp -i ~/.ssh/azure_rsa sqljdbc_6.0.8112.100_enu.tar.gz ci@teamcity.example.ca:/home/ci

# host
$ sudo docker cp sqljdbc42.jar teamcity:/data/teamcity_server/datadir/lib/jdbc









========
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

It looks like there's a powershell cmdlet for ssh now
http://www.thomasmaurer.ch/2016/04/using-ssh-with-powershell/
but I'm still using bash from cygwin.

// create a keypair
ssh-keygen -t rsa -f ~/.ssh/azure -C "hello+azure@example.org"

// log in to your server
ssh -i ~/.ssh/azure_rsa -p 2200 ci@teamcitymgmt.eastus.cloudapp.azure.com


## Add Azure Docker Volume Driver

It looks like ACS uses Ubuntu 14.04 by default, which means that
you will be using upstart. ...


When you edit azurefile-dockervolumedriver.default, the values can be found under the "storage account" that you created---the AF_ACCOUNT_NAME will be e.g. "teamcityzept" and the AF_ACCOUNT_KEY can be found under the Access Keys

Check the file `/var/log/upstart/azurefile-dockervolumedriver.log`
often, or do a `tail -f` on it becuase errors get reported here, not
on the console.

```bash
# uname -a
Linux swarm-master-6453DA0F-0 3.19.0-65-generic #73~14.04.1-Ubuntu SMP Wed Jun 29 21:05:22 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```




## Mount the azure file system 

 Error looking up volume plugin azurefile: legacy plugin: plugin not found



This means

cd /etc/init
wget https://raw.githubusercontent.com/Azure/azurefile-dockervolumedriver/master/contrib/init/upstart/azurefile-dockervolumedriver.conf

cd /etc/default
wget https://raw.githubusercontent.com/Azure/azurefile-dockervolumedriver/master/contrib/init/upstart/azurefile-dockervolumedriver.default -O azurefile-dockervolumedriver

cd /usr/bin
wget https://github.com/Azure/azurefile-dockervolumedriver/releases/download/v0.5.1/azurefile-dockervolumedriver




```
# docker volume create -d azurefile -o share=teamcityexample --name=logs
# docker volume create -d azurefile -o share=teamcityexample --name=data

```

check them:

```
# docker volume ls
DRIVER              VOLUME NAME
local               ...
azurefile           data
azurefile           logs

```

==============


On Azure:

Create an Azure Container Service with a slow machine

Create a Storage Account with some file shares

New -> Storage -> Create Storage Account

<figure>
 	<img src="/images/teamcity/create_storage_azure.png">
 	<figcaption>Create Storage Account</figcaption>
</figure>

Give it a name like "teamcityexample".  Once it's created (it will
take a minute) you'll need to open it, click on "Files" and
add two file shares, "logs" and "data"

<figure>
 	<img src="/images/teamcity/create_file_shares.png">
 	<figcaption>Create File Shares</figcaption>
</figure>

Then you'll need to go back to the main page for the
Storage Account and click on "Access Keys".  Take note of one of the
keys---you'll need it later on.




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

`/etc/default/azurefile-dockervolumedriver` will look like this 

```
AF_ACCOUNT_NAME=teamcityexample
AF_ACCOUNT_KEY=PUT_YOUR_STORAGE_ACCOUNT_KEY_HERE
```

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

docker run -it --name teamcity -e TEAMCITY_SERVER_MEM_OPTS="-Xmx2g -XX:MaxPermSize=270m -XX:ReservedCodeCacheSize=350m" -v /teamcity/data:/data/teamcity_server/datadir  -v /teamcity/logs:/opt/teamcity/logs -p 80:8111 jetbrains/teamcity-server 

docker run -d --name teamcity -e TEAMCITY_SERVER_MEM_OPTS="-Xmx2g -XX:MaxPermSize=270m -XX:ReservedCodeCacheSize=350m" -v data:/data/teamcity_server/datadir  -v logs:/opt/teamcity/logs -p 80:8111 jetbrains/teamcity-server 



Now go in and look at your load balancer---there should be a public IP
address attached to your "swarm-agent-pool".  You should be able 

<figure>
 	<img src="/images/teamcity/team_city_start.png">
 	<figcaption>TeamCity installation screen</figcaption>
</figure>

Let's make sure we got this right---let's start a bash shell on our `teamcity` container:

```
> docker exec -it teamcity bash
$ mount -l | grep teamcity
/dev/sda1 on /opt/teamcity/logs type ext4 (rw,relatime,discard,data=ordered)
/dev/sda1 on /data/teamcity_server/datadir type ext4 (rw,relatime,discard,data=ordered)
```


Add the JDBC driver:

[from here](https://www.microsoft.com/en-us/download/details.aspx?displaylang=en&id=11774)

```
scp -i ~/.ssh/azure_rsa sqljdbc_6.0.8112.100_enu.tar.gz ci@teamcitymgmt.eastus.cloudapp.azure.com:/home/ci
```
