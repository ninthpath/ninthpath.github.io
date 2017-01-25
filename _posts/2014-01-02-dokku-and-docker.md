---
layout: post
title: Dokku and Docker
---

A few months ago, I wanted to try out [Docker](https://www.docker.com/), so I installed [Dokku-alt](https://github.com/dokku-alt/dokku-alt) on a [DigitalOcean](https://www.digitalocean.com) VPS.  Dokku is basically Heroku on Docker.  You push your git repo to a repo on your Docker server, and Dokku will make a container for that app.  It’s easy to create containers and link them together, like Node+MariaDB.

The downside is that it’s annoying to customize.  Example: Dokku’s default Postgres container uses 9.1 and the encoding is SQL_ASCII.  If you want to use 9.4 and UTF8, you would have to dig through the plugin system, and manually edit them.  If you’re going to put in the time to read through the Dokku documentation and code, you might as well RTFM and use Docker proper.  I’m not the only one that feels this way.  Check out this thread on [Hacker News](https://news.ycombinator.com/item?id=9054503).  

I ended up using [Docker Compose](https://docs.docker.com/compose/), which makes it easy to create containers and link them together.  It doesn’t do it in one line, like Dokku, but the tradeoff is that you get to customize the containers.  Dokku also automatically creates the subdomain in nginx for your new service.  So if you use Docker Compose, you’ll have to manually edit the nginx.conf.
