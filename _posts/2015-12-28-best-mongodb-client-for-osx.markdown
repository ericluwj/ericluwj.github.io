---
layout: post
title:  "Best MongoDB Client for OSX"
description: "I was using the latest version of Robomongo (0.8.x), but it doesn't work if I needed to connect to my 0.3.0 MongoDB database."
date:   2015-12-28
---

<p class="intro"><span class="dropcap">W</span>ith the advent of MongoDB 3.x, we finally can look forward to performance and efficiency gains, especially with a new and better storage mechanism. However, due to a new authentication method being implemented as its default, it is breaking lots of existing MongoDB clients.</p>

I was using the latest version of Robomongo (0.8.x), but it doesn't work if I needed to connect to my 0.3.0 MongoDB database. It appears that if a database was created on a MongoDB 3.0 server, it will by default use the new SCRAM-SHA-1 challenge-response user authentication mechanism, which is still not supported by many of the GUI clients out there.

So apparently, many other MongoDB clients (e.g. UMongo, MongoHub) were not able to work as well. I was really disappointed. After all, it has already been 9 months since the introduction of MongoDB 3.0.1. And with Robomongo being considered the best MongoDB GUI client for OSX, I was wondering how it could possibly take so long for a patch.

Fortunately, I tried [MongoChef](http://3t.io/mongochef/) and it works. And the best thing is, its UI is actually pretty user friendly and is really similar to Robomongo. In fact, it's free for non-commercial use. If you were a Robomongo user, MongoChef would definitely be the best replacement tool.