---
layout: post
title:  "Installing & Using Docker in OS X for Beginners"
date:   2015-10-27
---

<p class="intro"><span class="dropcap">D</span>ocker is really a pretty useful tool, if you didn't know that. Never has it been easier to develop an application in a uniform environment, stage an application in a similar environment to production, and last but not least push the application up to production. So, let's start learning how to install and use Docker in OS X.</p>

##Installing Docker in OS X##

Download and install [Docker Toolbox](https://www.docker.com/docker-toolbox).
You may even want to have a quick crash course at [http://docs.docker.com/mac/started/](http://docs.docker.com/mac/started/).

The most important takeaway you have to understand is, you can't run Docker natively in OS X. So Docker provides the `docker-machine` tool to create and attach to a Docker Virtual Machine via VirtualBox.

##Using Docker in OS X##

This is the part that Docker does not communicate clearly.
Most of the time, users who just installed Docker would naturally be using it with a local VM.

`docker` has been installed in your system, but you can't simply use `docker` as it is, without setting the necessary docker environment variables.
The Docker Quickstart Terminal serves the purpose of starting the local VM if it does not yet exist or has not been started and then setting the docker environment variables to allow `docker` to connect to the Docker daemon on the local VM.

In other words, you can't just open another terminal shell and start using `docker`.
If you have already started the Docker Quickstart Terminal, we can be sure that the local VM has been setup.
But, you would need to manually set the docker environment variables with the following:

	eval "$(docker-machine env default)"

`default` actually refers to the name of the local VM started.
At this time of writing, the Docker Quickstart Terminal creates a VM called 'default'.

Otherwise, you would have been like me, wasting time figuring out why `docker` is not building or running and spouting some incomprehensible errors:

	Post http:///var/run/docker.sock/v1.20/build?cgroupparent=&cpuperiod=0&cpuquota=0&cpusetcpus=&cpusetmems=&cpushares=0&dockerfile=Dockerfile&memory=0&memswap=0&rm=1&t=ericluwj%2Fjobbies&ulimits=null: dial unix /var/run/docker.sock: no such file or directory.
	* Are you trying to connect to a TLS-enabled daemon without TLS?
	* Is your docker daemon up and running?

I think one of the questions that is apparently missing in the error message is whether you have set the docker environment variables.
So yes, please remember to **set the docker environment variables** before using `docker`.