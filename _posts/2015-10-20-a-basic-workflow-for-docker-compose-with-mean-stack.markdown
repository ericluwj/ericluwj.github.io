---
layout: post
title:  "A basic workflow for Docker (Compose) with MEAN stack"
description: "Interestingly, I found a lack of articles illustrating a basic workflow for Docker or Docker Compose with the MEAN stack, so I decided to provide some insight."
date:   2015-10-20
---

<p class="intro"><span class="dropcap">I</span>nterestingly, I found a lack of articles illustrating a basic workflow for Docker or Docker Compose with the MEAN stack, so I decided to provide some insight.</p>

For the uninformed, the MEAN stack refers to **MongoDB**, **ExpressJS**, **AngularJS**, **Node.js**.

I know you probably can find some docker files at the mean.io or mean.js repositories, but interestingly they are still pretty limited in terms of usage. What if you wanted for more than development? And besides at this time of writing, the mean.js docker files are not foolproof enough. The docker workflow for mean.js causes permission issues with additional plugins such as grunt-contrib-imagemin. As such, I have created my basic workflow on top of these files to provide for a more complete workflow using `docker` and `docker-compose`.

As a sidenote, my project stems from the Yeoman generator at https://github.com/DaftMonk/generator-angular-fullstack, which provides a much more practical project framework compared to mean.io and mean.js though.

### Development / Staging

So for the start, here are the 2 files for the complete development/staging workflow.
Just copy docker-compose.yml and Dockerfile into your project directory and run `docker-compose up` to get it started.

Note: I assume that you already know how to install `docker-compose` (https://docs.docker.com/compose/install/).

docker-compose.yml:
{% highlight yaml %}
app:
  build: .
  links:
   - db
   - redis
  ports:
   - "8080:8080" # change to whatever port is to be used
  environment:
   - NODE_ENV=development # production for staging
  env_file:
   - ./server/config/development.env # required only if you have environment settings to load
db:
  image: mongo
  ports:
   - "27017:27017"
# Optional: if you are using Redis too, otherwise remove this section.
redis:
  image: redis
  ports:
   - "6379:6379"
{% endhighlight %}

Dockerfile:
{% highlight yaml %}
FROM node:0.10

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

RUN npm install -g grunt-cli bower # add other tools that you use too, e.g. `gulp`

RUN groupadd -r node \
&&  useradd -r -m -g node node

COPY . /usr/src/app/
RUN chown -R node:node /usr/src/app

USER node
RUN npm install \
 && grunt build # required for staging

ENV NODE_ENV development # production for staging
CMD [ "npm", "start" ]
EXPOSE 8080 # change to whatever port is to be used
{% endhighlight %}

### Production

Just as a sidenote, you need to understand that `docker` is meant to function for a single host, so this workflow will only work on a single machine. If you need to run distributed applications on separate machines, that will be a lot more troublesome, so we'll have that discussion another day.

Do note that the directories if necessary should be created on the host machine before `docker-compose` is run.

To get the docker image built with `docker` easily up onto the production machine, let's use [Docker Hub](https://hub.docker.com/).
This is much easier than trying to create your own Docker Registry and then trying to wrap it up with TLS. The Docker daemon is really unstable, and all my attempts to incorporate TLS failed terribly.
Create an account at [Docker Hub](https://hub.docker.com/) and then create a repository (public/private), e.g. app.
If you named it as "app", the image will be named *username*/app, where the username is your Docker Hub username.

To start, build the application to prepare to be transferred over to production.
Use the following Dockerfile to build.

Dockerfile:
{% highlight yaml %}
FROM node:0.10

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

RUN npm install -g grunt-cli bower # add other tools that you use too, e.g. `gulp`

RUN groupadd -r node \
&&  useradd -r -m -g node node

COPY . /usr/src/app/
RUN chown -R node:node /usr/src/app

USER node
RUN npm install \
 && grunt build # run commands to build everything

WORKDIR /usr/src/app/dist # change to distribution folder
ENV NODE_ENV production
CMD [ "npm", "start" ]
EXPOSE 8080 # change to whatever port is to be used
{% endhighlight %}

Run the following command in the project directory.

    docker build -t="username/app" .

If you have not logged in to Docker Hub yet, do this and type in your credentials/details:

    docker login

Now upload the built docker image to the remote docker repo at Docker Hub.

    docker push username/app


Now onto the production server.
You will need to install `docker` and `docker-compose`.
Prepare the necessary directories on the production server as stated as volumes in the docker-compose.yml file.
Similarly, do a `docker login` to login to Docker Hub.
You will need to have the docker-compose.yml file and production.env file containing your production environment variables.
Run `docker-compose up -d` to start running all the docker containers in detached mode.

docker-compose.yml:
{% highlight yaml %}
web:
  image: jwilder/nginx-proxy
  ports:
   - "80:80"
   - "443:443" # optional for HTTPS
  volumes:
   - /var/run/docker.sock:/tmp/docker.sock:ro
   - ~/certs:/etc/nginx/certs # "~/certs" needs to be created on host to store SSL certificates (if necessary)
app:
  image: username/app
  links:
   - db
   - redis
  ports:
   - "8080:8080"
  environment:
   - NODE_ENV=production
   - VIRTUAL_HOST=www.example.com
  env_file:
   - production.env # this would be useful on production as you set production variables only on the production machine
db:
  image: mongo
  ports:
   - "27017:27017"
  volumes:
   - ~/data/mongo:/data/db # "~/data/mongo" needs to be created on host to store MongoDB data
# Optional: if you are using Redis too, otherwise remove this section.
redis:
  image: redis
  ports:
   - "6379:6379"
  volumes:
   - ~/data/redis:/data # "~/data/redis" needs to be created on host to store Redis data
{% endhighlight %}

Once you have done all these steps, note how relatively easy it is to push your updated application to production.
Just use `docker` to rebuild the app image, push the image up to the repo, use server to pull image from repo and use docker to rerun the image.

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
