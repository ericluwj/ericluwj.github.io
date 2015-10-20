---
title:  "Installing Docker Compose in CoreOS"
date:   2015-10-20 11:16:00
description: We will provide the instructions on how to install docker-compose in CoreOS
---

With the acquisition of Fig, it is really cool now that Docker has a new tool `docker-compose` that allows you to do `docker` to multiple docker containers all at once.

Can Docker Compose be used in CoreOS?
It is interesting that most people believe that Docker Compose can't be installed in CoreOS because it has a readable-only user filesystem.
But the answer is, **YES**, Docker Compose can be installed and then used in CoreOS.

So here are the instructions.
Prepare a CoreOS machine first.

Enter root mode:

    sudo su -

Create the necessary writable directories for installation of `docker-compose`:

    mkdir /opt/
    mkdir /opt/bin

Download/install `docker-compose`:
You may replace "1.4.0" with any version of `docker-compose` you would like to install.

    curl -L https://github.com/docker/compose/releases/download/1.4.0/docker-compose-`uname -s`-`uname -m` > /opt/bin/docker-compose

Make the `docker-compose` binary executable.

    chmod +x /opt/bin/docker-compose

Exit root mode:

    exit

That's it. You may now use `docker-compose` directly in the command line.