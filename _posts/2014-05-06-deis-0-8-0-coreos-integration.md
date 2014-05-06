---
layout: post
title: Deis 0.8.0 - CoreOS Integration
author: mboersma
meta:
  - name: description
    content: Deis integrates CoreOS in version 0.8.0
  - name: keywords
    content: deis, release, 0.8.0
---

The Deis project is proud to announce our v0.8.0 release. Deis now runs on CoreOS and no longer requires Chef. Apps deploy to clusters of machines using a simpler, faster and more reliable job scheduling interface.

If you are updating from Deis version 0.7.0 or earlier, read the [MIGRATING.md](https://github.com/opdemand/deis/blob/master/MIGRATING.md) document for details.

<!--more-->

## What is Deis?

Deis is an open source PaaS that makes it easy to deploy and manage applications on your own servers. Deis builds upon Docker and CoreOS to provide a lightweight PaaS with a Heroku-inspired workflow.

## 0.8.0 Summary

### New Features

- **Cluster-wide scheduling** Deis itself and apps it deploys use the [`fleet`](https://coreos.com/docs/launching-containers/launching/launching-containers-fleet/) distributed init system for flexible scheduling across available machines.

- **Basic integration test** Test your local changes, or verify your new Deis installation with the basic [integration tests](https://github.com/deis/deis/tree/master/test) we've begun in the `test/` directory.

- **Doc updates** We added new content appropriate to our CoreOS changes to Deis' [documentation](http://docs.deis.io/), and put README.md files in each top-level directory to help you get oriented.

- **`deis create --no-remote`** allows creating a Deis app without adding a git remote.

- **New deis/logger** New, lightweight syslogd server written in Go to handle aggregation of application logs.

- **[deis/helloworld](https://github.com/deis/helloworld)** is a new Dockerfile-based example app to help you get started.

- **#deis at botbot.me** Joining the [#deis](https://botbot.me/freenode/deis/) conversation on IRC is even easier thanks to the excellent service from [botbot.me](https://botbot.me/freenode/deis/).

- **easy-fix tag** Help the Deis revolution by picking an [easy-fix issue](https://github.com/deis/deis/issues?labels=easy-fix&state=open) to start contributing.

### Under the Hood

- Moved project to https://github.com/deis/deis
- Updated to Docker 0.10.0 (deis/builder)
- Updated to Django 1.6.4 (deis/controller)
- Updated to CoreOS 298.0.0
- Implemented finite state machine for containers
- Improved scheduling unit tests and error handling
- `deis ps` is preferred to `deis containers`
- Removed unused cloud provider libraries from `pip install`

### Bug Fixes

- `deis rollback` works again
- fixed 404 on `deis logs`
- fixed coveralls.io code coverage report and badge
- isolated PostgreSQL test suite errors on Travis-CI.org to one (skipped) method
- filled a gap in available ephemeral port range for apps

### Community Shout-Outs

We want to thank the following Deis community members for creating GitHub issues, providing support to others, and working on various Deis branches:

* @dginther - user-data fix to stop update-manager service
* @goloroden - Vagrant NFS / root access issue
* @gshiva - EC2 testing, bug reports
* @hyao - testing, nfs-kernel-server doc note
* @johanneswuerbach - testing, registry bug, S3, cache, other reg options PRs
* @jschneiderhan - missing Docker repositories issue
* @jwaldrip - nginx welcome page issue, MySQL question, bug finding
* @lukaso - found `fleetctl` command doc issue
* @philiptzou - Vagrant static files in Django issue
* @recipher - EC2 testing
* @zyegfryed - remove duplicate `etcdctl` install statement

And a big hello to kristjanpk, nathansamson, StDiluted, stefanfoulis, and all the others who keep our IRC channel so lively!

## Known Issues

### Container Timeouts
We are still seeing stability issues with the public Docker Index and its content delivery network.  When bootstrapping a Deis cluster, this can manifest as timeout errors, missing images and/or missing layers.  Manual `systemctl` commands will usually resolve the issue.  We are working with the Docker team on the root issue.

### Proxy for Platform Services
As part of moving Deis into Docker containers, we had to change the exposed ports for some core platform services.  For example, the Django API server is now exposed on 8000/tcp rather than 80/tcp.  We will soon update the Deis router to [expose platform services](https://github.com/opdemand/deis/issues/535) on standard ports.

## What's Next?

### Deis Upgrades

In our push toward 1.0, our most important priority is making sure Deis upgrades are fast, painless and backwards compatible.  We are devoting significant resources to this effort, and are working closely with the CoreOS team who has a great deal of experience with rolling upgrades.

### Better Heroku Compatibility

We want to ensure Deis follows Heroku semantics as closely as possible.  To achieve this, we may need to make changes to the Deis client, REST API as well as new Heroku-provided features deemed necessary for Deis.  We plan to have the bulk of this compatibility implemented in the next release.

### Improved Docker Integration

We have heard from the community and our customers that Deis needs a way to deploy and promote existing Docker images outside of a `git push` style build process.  This is critical for implementing CI/CD pipelines on top of Deis.  We are currently investigating the best workflow in conjunction with our friends at Docker HQ.  We hope this new feature, combined with enhanced support for pushing `Dockerfile` repositories, will provide the best Docker-native experience of any PaaS on the market.

### Platform Security & HA

With Deis being used for more and more real-world deployments, we need to address remaining [security](https://github.com/deis/deis/issues?labels=security&state=open) issues as well as [high-availability of the platform itself](https://github.com/deis/deis/issues/874).  This work has already begun in earnest, and will be a prerequisite for our 1.0 release.

## Future

### Service Gateway
We need to make it as easy as possible for ops folks to publish a set of reusable backing services (databases, queues, storage, etc) and allow developers to attach those services to applications.  This will be done in a loosely coupled way, following Twelve Factor best practices.  You can review the initial implementation and follow progress [on this GitHub issue](https://github.com/opdemand/deis/issues/231).

### Interactive `deis run`
Though we provide the ability to run admin commands inside containers, we don't currently support interactive shells into containers (i.e. `deis run bash`).  Once this infrastructure is in place, this will also allow us to implement log tailing and other real-time features.

## How can you help?

* Star our [GitHub repository](https://github.com/opdemand/deis)
* Help spread the word about [@opendeis](http://twitter.com/opendeis) on Twitter
* Explore contributing to the Deis project by joining the #deis channel on Freenode

You can learn about other ways to [get involved](http://deis.io/get-involved/) on our website.
