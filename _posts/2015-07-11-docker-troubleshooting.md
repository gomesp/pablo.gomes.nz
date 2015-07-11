---
layout: post
title:  "Troubleshooting Docker after upgrading"
date:   2015-07-11
categories:
---
After upgrading docker from v1.6.1 to v1.7.0 I got a few errors. These are the steps followed to recover from the error.

## Problem 1: "Are you trying to connect to a TLS-enabled daemon without TLS?"

Right after the upgrade, I tried running hello-world, and got the "Are you trying to connect to a TLS-enabled daemon without TLS?" error message.

The solution to my first problem was to delete and re-create the boot2docker vm.

{% highlight sh %}
# First, checking my current version.
$ boot2docker -v
Boot2Docker-cli version: v1.6.1
Git commit: 076b58d

# OK, so it was time to upgrade.
$ boot2docker upgrade
Backing up existing docker binary...
# ...
Success: downloaded https://github.com/boot2docker/boot2docker/releases/download/v1.7.0/boot2docker.iso
	to ~/.boot2docker/boot2docker.iso

# So far the operation is a success.
$ boot2docker -v
  Boot2Docker-cli version: v1.7.0
  Git commit: 7d89508

$ docker version
  Client version: 1.7.0
  Client API version: 1.19
  Go version (client): go1.4.2
  Git commit (client): 0baf609  

# Let's test the new version.
$ boot2docker up
Waiting for VM and Docker daemon to start...
....................oo
Started.

# Time to run the good old hello-world
$ docker run hello-world
Post http:///var/run/docker.sock/v1.19/containers/create: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?

# OK, that didn't go so well. Let's clean things up.
$ boot2docker delete && boot2docker init
{% endhighlight %}

## Problem 2: "dial tcp 192.168.59.103:2376: i/o timeout"

After recreating the boot2docker vm, I started it and tried running hello-world again. Now docker was timing out. For some reason my boot2docker vm had a different IP address than docker thought it did.

I found the solution in this [boot2docker github issue](https://github.com/boot2docker/boot2docker/issues/392#issuecomment-62318587), which was to:
* Identify the Host-only network adapter used by the boot2docker Virtualbox vm;
* Delete the network adapter;
* Delete the boot2docker vm and initialise it again.

{% highlight sh %}
# Wait for Virtualbox to finish...
$ boot2docker up
Waiting for VM and Docker daemon to start...
..........................oooooooooooooooooooooo
Started.
Writing ~/.boot2docker/certs/boot2docker-vm/ca.pem
Writing ~/.boot2docker/certs/boot2docker-vm/cert.pem
Writing ~/.boot2docker/certs/boot2docker-vm/key.pem
Your environment variables are already set correctly.

# Let's try hello-world again.
$ docker run hello-world
An error occurred trying to connect: Post https://192.168.59.103:2376/v1.19/containers/create: dial tcp 192.168.59.103:2376: i/o timeout

# Checking my IP. No wonder I get a timeout error.
$ boot2docker ip
192.168.59.104

# Go to VirtualBox (menu) => Preferences => Network => Host Only Networks => delete network adapters
$ boot2docker delete && boot2docker init

# Success at last.
$ docker run hello-world
Hello from Docker.
This message shows that your installation appears to be working correctly.
{% endhighlight %}
