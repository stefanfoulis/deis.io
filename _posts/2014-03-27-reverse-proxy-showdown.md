---
layout: post
title:  Reverse Proxy Showdown
date:   2014-03-27
author: bacongobbler
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
[dynamic configuration][1] through a Lua extension! Nginx is also documented out the
wazoo, which is another huge plus.

One interesting point about Nginx is that it does not rely on threads to handle requests
(I'm looking at you, Apache). Inbound requests are handled through an event-driven
architecture, allowing it to scale and solve the [C10K problem][2]. This architecture
is ridiculously scalable due to the low memory footprint and event-driven triggers to
handle anything from a small blog post (like this one!) to CloudFlare's gargantuan
Content Delivery Network.

There's also a bundled version of nginx called openresty. openresty is a full-fledged web
application server by bundling the standard nginx core, lots of 3rd-party nginx modules,
as well as most of their external dependencies. It makes dependency management for
third party modules a lot easier than bundling it yourself.

## To be Continued

Our next blog post will look into the different proxies presented to us, as well as a
proof of concept that is containerized using Docker, dynamic configuration from [etcd][3],
and advanced templating features from [confd][4].

[1]: http://sosedoff.com/2012/06/11/dynamic-nginx-upstreams-with-lua-and-redis.html
[2]: http://www.kegel.com/c10k.html
[3]: https://github.com/coreos/etcd
[4]: https://github.com/kelseyhightower/confd
