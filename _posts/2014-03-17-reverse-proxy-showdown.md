---
layout: post
title:  Reverse Proxy Showdown
date:   2014-03-17
author: mattf
tags:   proxy nginx lighttpd pound hipache strowger comparison
comments: true
---

![Containerize all the things!](/assets/img/containerize-all-the-things.jpg)

Since our v0.5.2 release, Deis has been completely containerized. Every component of Deis
has been packaged and made available as a trusted build on
[Docker's public registry](https://index.docker.io/u/deis/). This is one small step
towards the larger goal of `docker run deis/deis`; a container that contains every
component of the deis controller in one Docker image. One thing we overlooked was the
proxy, so I had to come up with a solution that would meet Deis' demands. This post
explains our research on different proxies and why we decided on choosing nginx as our
default router.

We accidentally left out one component when containerizing all the things: a reverse
proxy. After the release, we decided to start doing some research on the different
proxies and to come up with a solution for our use case. Of all the proxies available, we
decided to look at 6 different proxies. After some filtering, we set our eyes on 1
proxy, containerized it, and then ran some more tests against it for your enjoyment (and
ours). Here's the list of the proxies we looked at:

- Nginx: a high performance, open source web application accelerator
- Lighttpd: a secure, fast, compliant, and flexible web-server for high-performance environments
- Pound: a reverse proxy, load balancer and HTTPS front-end for Web server(s)
- Hipache: a distributed HTTP and websocket proxy developed by Dotcloud.
- Strowger: [flynn.io](https://flynn.io/)'s HTTP/TCP cluster router
- HAProxy: a fast and reliable solution offering high availability, load balancing, and proxying

We also had some requirements that we were keeping in mind when we were working with
these proxies:

- Does it possess the ability to add/remove upstreams without downtime?
- Does it have SSL support?
- Does it have TCP support?
  * We need this for SSH and other non-HTTP connections (websockets, for example)
- How well does it perform under heavy loads?
- What is its license?
  * AGPL is a deal-breaker, for example
- How large is the community?
  * Visibility
  * Mailing Lists
  * Stack Overflow
  * Documentation

With that, I came up with the following comparison table. From this data, we decided to
look at four different solutions a little closer: Strowger, HAProxy, Hipache, and Nginx.

              | Nginx        | Lighttpd     | Pound | Hipache | Strowger | HAProxy
:------------ | :----------: | :----------: | :---: | :-----: | :------: | :-----:
Zero Downtime | :+1:         | :+1:         | :+1:  | :+1:    | :+1:     | :+1:
SSL Support   | :+1:         | :+1:         | :+1:  | :+1:    | :+1:     | :-1:
TCP Support   | :+1:         | :-1:         | :-1:  | :-1:    | :-1:     | :+1:
Prod-ready?   | :+1:         | :+1:         | :+1:  | :+1:    | :-1:     | :+1:
License       | 2-clause BSD | 2-clause BSD | GPL   | MIT     | MIT      | GPLv2
Language      | C            | C            | C     | Node.js | Golang   | C

## Strowger

Strowger is the Flynn HTTP/TCP cluster router. It relies on service discovery to keep
track of what backends are up and acts as a standard reverse proxy with random load
balancing. Because of the reliance on service discovery for configuration, it is the only
reverse proxy capable of hot-swapping configuration without having to fork a new process.
Also, since all instances of Strowger get the same configuration, this makes it really
easy to start booting up more containers for redundancy. It's also written in Go.
However, because it is still a prototype in alpha, there are many unfinished features with
the software. They even
[recommend using HAProxy](https://github.com/flynn/strowger/tree/2a228596d142cf01bf443d1ffb3aede8df0f9e5f#benefits-over-haproxynginx)
for production use in their documentation. They're also
[missing TCP support](https://github.com/flynn/strowger/issues/9) as of this blog post,
which is a non-starter for us. Overall, we are excited to hear what will happen to
strowger in the future, but right now we need production-ready software.

## HAProxy

HAProxy is particularly suited for web sites crawling under very high loads while needing
persistence or Layer7 processing. It has never crashed in a production environment. Ever.
Its documentation is also not that bad, but nowhere near as stupendous as nginx's docs.
Did we mention that it's also never crashed in production? However, its lack of SSL
support and dynamic configuration puts a shoehorn in this solution.

## Hipache

Hipache is Docker Inc's (formerly known as Dotcloud) distributed HTTP and websocket proxy.
It is designed to route high volumes of http and websocket traffic to unusually large
numbers of virtual hosts, in a highly dynamic topology where backends are added and
removed several times per second. It is particularly well-suited for PaaS
(platform-as-a-service) and other environments that are both business-critical and
multi-tenant. Hipache is developed in Node.js and is based on the node-http-proxy library;
a library that we know and are comfortable with. Regardless, Hipache lacks TCP support,
which is a non-starter for us.

## Nginx

Nginx is a free, open-source, high-performance HTTP server and reverse proxy, as well as
an IMAP/POP3 proxy server. It is known for high performance, stability, rich feature
set, simple configuration, and low resource consumption. Nginx also powers several
high-visibility sites, such as Netflix, Hulu, Pinterest, CloudFlare, GitHub, Heroku,
RightScale, Engine Yard and MaxCDN, to name a few. It has proxy support for SSL, TCP,
and websockets, which passes our initial tests with flying colours. It even has
[dynamic configuration][dyn-config-nginx] through a Lua extension! Nginx is also
documented out the wazoo, which is another huge plus.

One interesting point about Nginx is that it does not rely on threads to handle requests
(I'm looking at you, Apache). Inbound requests are handled through an event-driven
architecture, allowing it to scale and solve the [C10K problem][c10k]. This architecture
is ridiculously scalable due to the low memory footprint and event-driven triggers to
handle anything from a small blog to CloudFlare's gargantuan Content Delivery Network.

## Proof of Concept

Because no other proxy supported the big three (HTTP, SSL, and TCP) all at once, nginx
wins by default! In order to come up with a proof of concept to see if nginx would work
in our environment, I built a containerized version of the router, which is available
to view [here](https://github.com/opdemand/deis/commit/660af5d9a02aa3b1ef26aaa66b00dfdd08114201).
Let's break down the component:

### Dockerfile

The Dockerfile does one thing and one thing only: build nginx with Lua support and a
redis connection library for Lua. To do that, we compile nginx from source with Lua
already installed into the image.

### Confd

[confd](https://github.com/kelseyhightower/confd) is a new and exciting project based on
two things that we have fallen in love with over here: etcd and Go. confd is a lightweight
configuration management tool that allows you to template any file and update those
templates on the fly, based on certain changes in any of the etcd keys your template
relies on. In this example, we want to template our nginx configuration file so that users
can dynamically change the connection string to our Redis backend at any point. When you
change the key's value in etcd, the value in nginx.conf is changed the next time it
synchronizes with confd, which is about every 5 seconds. It's pretty powerful stuff.

### Lua/Redis Dynamic Configuration

This little gem really knocked really blew me out of the water. The knowledge all comes
from [this blog post][dyn-config-redis], so the thanks goes out to Dan Sosedoff. This
powerful snippet allows us to dynamically set and unset nginx upstreams with zero
downtime, allowing us to change values in Redis freely without interruption to existing
connections or without any hiccups in new connections. In fact, it's so fast that the
request/response time hardly feels anything due to how blazingly fast Redis works! For
now, it's just a proof of concept, but we can do some pretty cool stuff with it:

    $ dig registry.deisapp.com
    [...]
    ;; ANSWER SECTION:
    registry.deisapp.com. 1649 IN  A   107.170.227.113
    $ dig deisapp.com
    [...]
    ;; ANSWER SECTION:
    deisapp.com.   1800    IN  A   107.170.227.113
    $ ssh -i ~/.ssh/deis-controller root@deisapp.com
    root@deis-controller-t1LHh:~# redis-cli
    redis 127.0.0.1:6379> KEYS *
    1) "deisapp.com"
    2) "registry.deisapp.com"
    redis 127.0.0.1:6379> GET registry.deisapp.com
    "107.170.227.113:5000"
    redis 127.0.0.1:6379> exit
    root@deis-controller-t1LHh:~# curl -L registry.deisapp.com
    "docker-registry server (dev)"
    root@deis-controller-t1LHh:~# redis-cli
    redis 127.0.0.1:6379> DEL registry.deisapp.com
    (integer) 1
    redis 127.0.0.1:6379> exit
    root@deis-controller-t1LHh:~# curl -L registry.deisapp.com
    <html>
    <head><title>404 Not Found</title></head>
    <body bgcolor="white">
    <center><h1>404 Not Found</h1></center>
    <hr><center>nginx/1.0.15</center>
    </body>
    </html>

If you intentionally kill deis/cache (the current Redis backend that the router is using),
then the server returns an HTTP 500 error. We can completely lock down all of Deis'
components and just have a couple entries in Redis to route to those components through
nginx.  As a proof of concept, it seems to work pretty well, and we're really excited with
what we can do with this new component.

## Conclusion

After comparing the three, we came down to only Nginx as a solution for our needs. While
HAProxy and Hipache are stable, it does not have SSL support, which is a non-starter.
Strowger is not yet ready for production, and we need a product that works now. Nginx wins
by default!

[c10k]: http://www.kegel.com/c10k.html
[dyn-config-redis]: http://sosedoff.com/2012/06/11/dynamic-nginx-upstreams-with-lua-and-redis.html
