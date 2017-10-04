---
layout: article
title: Wormhole!  Tunnel a Firewalled Machine Onto the Web via Azure
tags: [ssh reverse tunnel autossh teamcity azure firewall dotnet-core]
categories: articles
comments: true
excerpt: Make a web site that exists behind a firewall (or other service) available to the internet.
image:
  teaser: teamcity/wormhole-400x250.png
ads: true
---

*October 4, 2017*

>
> This is the first part of a series of blog posts on DevOps for **DotNet Core**, **TypeScript** and **React**.
>

This spring I set up a Continuous Deployment pipeline on TeamCity for a DotNet Core / TypeScript / React 
application which deploys to Azure.  It would have been somewhat expensive to host this CD setup in the 
cloud, considering all the old unused hardware I have sitting in the office.  I didn't want to fiddle with the 
firewall to put these machines online, but fortunately it's fairly easy to set up a tunnel to the outside
without needing to touch the firewall—and that's what I'll be describing here.  I have 
Raspberry Pis which sit behind my ISPs' firewalls that I've had to make accessible, so I set this up 
exactly the same way.  You can apply this technique to access all sorts of firewalled services without 
needing to do any local network configuration at all, like controlling the heat in a remote 
office à la [Mr. Robot](http://www.imdb.com/title/tt4158110/). 

<figure>
 	<img src="/images/teamcity/MrRobot.png">
 	<figcaption>Circumventing the Firewall</figcaption>
</figure> 

Here's how this will work:

- An *Office Server* will sit behind a firewall and serve HTTP responses.
- A *Proxy Server* will sit in the cloud and forward incoming HTTP traffic to the Office Server.
- An account called "autossh" will exist on on each of the two servers.
- The autossh account on the Office Server will create and maintain a "Reverse SSH Tunnel" 
by logging into the corresponding autossh account on the Proxy Server and linking a port on the 
Office Server to a port on the Office Server.  This tunnel is like a wormhole in Star 
Trek---internet traffic that goes into port X on the Proxy Server comes out on port Y on the 
Office Server.
- Azure will route IP traffic to the Proxy.

<figure>
 	<img src="/images/teamcity/wormhole-400x250.png">
 	<figcaption>An SSH Tunnel</figcaption>
</figure> 
 
If you're a Windows administrator who's fairly new to Linux, I've added some extra description which I hope
will reduce some of the cognitive load.
 
### The Office Server

Even though I develop C# and TypeScript on Windows, it's now easy to compile and test my code 
on Linux, so the first step was to install my favourite Linux distribution [Mint](https://linuxmint.com/) on 
one of my old machines.  If you haven't used Linux before, Mint has the advantage of being a
pretty standard Ubuntu distribution, but with a familiar desktop.  I'll call this the
"Office Server", and it sits behind the firewall.

I'll write about how to set up TeamCity for DotNet Core on Linux in a future post, but for now 
I will just assume that you already have a web site set up on your Office Server listening to
port 8080.

### The Proxy Server

The next step is to create a "Proxy Server" somewhere in the cloud.  We'll open port 80 on this server, and 
its job will be to push all HTTP traffic into the tunnel so that it arrives at our Office Server.  I 
have a proxy server at [Digital Ocean](https://www.digitalocean.com/) which is dirt-cheap to 
maintain, but for ths blog post I will be describing my Azure setup.

I chose an Ubuntu 16.04 LTS VM in Azure.  The slowest, cheapest VM they have is enough for my
purposes, so I'm using a "Standard A0" machine.  Boot this up, and log into it via SSH.

<figure>
 	<img src="/images/teamcity/create_ubuntu_azure.png">
 	<figcaption>Create an Ubuntu VM</figcaption>
</figure>

### Set up AutoSSH

Log in to both the Office Server and the Proxy Server, e.g.:

<pre>
$ ssh -i ~/.ssh/my_azure_rsa.pub myazureuser@1.2.3.4
</pre>

On each machine, create an "autossh" user.  Since there shouldn't be user activity on ths
account after we set it up, we'll turn the shell off by setting it to `/bin/false`.  We'll 
create the account and log in as autossh in the same way on each server like this:

<pre>
$ sudo useradd -m -s /bin/false autossh
$ sudo su - -s /bin/bash autossh
</pre>

That second line means "log in as autossh using the bash shell, and do it via the root account".  (Since
we turned the shell off, we need to explicitly specify that we want to use a bash shell for this session.)  You
should now have two open terminals, one for each server.

On the Office server, generate an ssh keypair:

<pre>
$ ssh-keygen
</pre>

Just use the defaults and don't set a passphrase.  Copy the `~/.ssh/id_rsa.pub` public key file that you 
just generated from its location on the Office Server to the Proxy Server.  You need to be careful with the
permissions---it will go in the same `~/.ssh` directory (the squiggle means "my home directory").  If 
you have to create it, make sure the directory has the permissions 700. (`chmod 700 ~/.ssh`).  In order
for this to work, you'll also need to copy `id_rsa.pub` to the file `authorized_keys` on the Proxy:

<pre>
$ mkdir 
$ cd ~/.ssh
$ cp id_rsa.pub authorized_keys
</pre>

At this point, you should be able to connect from your office server to your proxy server:

<pre>
$ ssh autossh@1.2.3.4
</pre>

You won't be able to log in (since you have no shell configured), but you should see "Welcome to Ubuntu"
before you get logged out---this means SSH is configured correctly.  If it isn't, try adding the `-v` flag
to the `ssh` command to debug what's going wrong.
 
Back on the office server, you'll need to install *autossh*.  This is the tool that keeps the tunnel alive.

<pre>
$ sudo apt-get update
$ sudo apt-get install autossh
</pre>
 
We're almost done with the Office server---the magic command to set up a tunnel is this:

<pre>
autossh -M 0 -q -f -N -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3" -R 8080:localhost:8080 autossh@1.2.3.4
</pre>
 
This is an alphabet soup of flags, which you can [look up](https://linux.die.net/man/1/autossh), but 
in general:

- `-M 0` - turn port monitoring off.  You can monitor a port if you want, but I haven't found that I need to bother.
- `-f` - run in background 

The other flags are passed to SSH:

- `-N` - don't execute a shell command via ssh, just forward the port
- `-q` - quiet mode
- `-o ...` - pass some options
- `-R X:localhost:Y` - create a tunnel the localhost port `X` on the Proxy server to port 'Y' Office Server.

At this point, you should be able to access the tunnel from the proxy server.  On the Proxy Server, you
can test this e.g.:

<pre>
$ curl http://localhost:8080
=> some kind of http response
</pre>

Now we need establish the route that the IP traffic will take through the Proxy server.  Set these 
lines in `/etc/sysctl.conf`:

<pre>
net.ipv4.ip_forward=1
net.ipv4.conf.all.route_localnet=1
</pre>

To activate these new rules in the kernel, type `sudo sysctl -p`.

In order to access our pipeline externally, we will route external traffic from port 80 on the
Proxy Server to localhost on port 8080.  This command will connect our SSH tunnel to the
outside world:

<pre>
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 127.0.0.1:8111
</pre>

At this point you should be able to access it from the Proxy Server, but using your external IP address, e.g.

<pre>
$ curl http://1.2.3.4
=> some kind of http response
</pre>

You can optionally enable access via local host:

<pre>
$ sudo iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-port 8111
$ curl http://localhost
=> some kind of http response
</pre> 

At this point, your two servers should be communicating, but the iptables and autossh commands
will be lost on reboot.  On the Proxy Server, you could use something fancy like `iptables-save` to
persist the iptables settings, but I generally don't bother---I just add the two iptables commands 
into `/etc/rc.local` which runs on boot.  Similarly, I make `autossh` run at boot time by adding 
this to `/etc/rc.local` on the office server:

<pre>
sudo su -s /bin/sh autossh -c 'autossh -M 0 -q -f -N -o "ServerAliveInterval 60" \
    -o "ServerAliveCountMax 3" -R 8080:localhost:8080 autossh@1.2.3.4'
</pre>

### Azure

The last thing to do is to configure Azure.  Booting an Ubuntu instance creates 
a "Network Interface" and an "NSG" or Network Security Group for you.  You just
need to 

- Set the Network interface's IP address to "Static"
- Configure the NSG to allow traffic to port 80

<figure>
 	<img src="/images/teamcity/config_network.png">
 	<figcaption>Configure Azure's Firewall</figcaption>
</figure>

(I also have port 443 configured.)


With luck, your web site should now be accessible to the web.  If you reboot both machines, they
should also automatically re-establish the connection.

Coming up in this series: *Set up TeamCity for DotNet Core*; *Continuous Deployment of DotNet Core 
to Azure via GitHub*.



