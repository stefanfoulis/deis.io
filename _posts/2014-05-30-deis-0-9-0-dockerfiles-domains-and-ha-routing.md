---
layout: post
title: Deis 0.9.0 - Dockerfiles, Domains, and HA Routing
author: mboersma
meta:
  - name: description
    content: Deis integrates routers, adds app domains, and improves Dockerfile support in 0.9.0
  - name: keywords
    content: deis, release, 0.9.0
---

The Deis project is excited to announce our v0.9.0 release. We've improved the
workflow for both Dockerfile and Procfile projects, added support for custom
app domains, and allowed for multiple routers to improve availability.

The Deis controller is now accessible at the hostname "deis" in your cluster's
domain at port 80. If you used to connect to `http://local.deisapp.com:8000`,
for example, now use `http://deis.local.deisapp.com`.

If you are coming from an earlier version of Deis, read the
["Upgrading Deis"](http://docs.deis.io/en/latest/operations/upgrading-deis/)
documentation for details.

<!--more-->

## What is Deis?

Deis is an open source PaaS that makes it easy to deploy and manage applications
on your own servers. Deis builds upon [Docker](http://docker.io/) and
[CoreOS](https://coreos.com/) to provide a lightweight PaaS with a
Heroku-inspired workflow.

## 0.9.0 Summary

### New Features

- **Improved dockerfile and procfile workflow** See
  https://github.com/deis/deis/pull/967 for more details
- **Heroku compatibility** is much improved, with CLI additions and better
  buildpack support
- **Proxy builder and controller** All Deis' components and hosted apps are now
  fronted by one or more routers
- **`nse deis-(component)`** lets you enter a running container on a CoreOS
  machine for debugging
- **Use rsync by default** NFS in Vagrant was problematic for many users, so we
  now default to `vagrant rsync`
- **`git push deis mybranch`** Deis allows pushing to branches other than master
- **`deis domains`** allows creating and binding custom domains to Deis apps
- **user registration toggle** A Deis cluster can now disallow new users from
  registering by changing an etcd key
- **gzip configuration settings** for routers can now be customized with etcd
- **message of the day** now greets users with a colorful Deis banner when
  logging in to a cluster machine
- **bare metal docs** for provisioning Deis and CoreOS on your own hardware were
  added to the contrib folder
- **integration tests** We've begun beefing up our test infrastructure at
  http://ci.deis.io/ to improve software quality and speed development

### Under the Hood

- Updated to current alpha CoreOS, 324.2.0
- Updated to Docker 0.11.1
- Updated to fleet 0.3.2

### Bug Fixes

- Fixed some Makefile targets that assumed a vagrant environment
- Always `docker pull` by specific tag to save bandwidth
- Repond with 204, not 404, when `deis logs` has nothing to show
- Close DB connections in threaded Django test execution
- Destroy app containers when destroying their cluster
- Announcers detect first exposed port instead of hardcoding to 5000
- Use btrfs filesystem inside builder
- `deis` CLI returns non-zero on all errors
- name containers by app and remove naming uniqueness constraint
- set etcd keys safely in /bin/boot
- many updates to deis/example-* apps
- warn properly if `FLEETCTL_TUNNEL` is unset

### Community Shout-Outs

We want to thank the following Deis community members for creating GitHub
issues, providing support to others, and working on various Deis branches:

* @aledbf - remove vagrant SSH dependency, registration_enabled bug find
* @andyshinn - testing, GCE and AWS documentation gists
* @dudymas - registration_enabled bug find
* @danscan - Dockerfile / AWS testing, app deploy issue, envvars in builder
* @dobrite - doc reviewing, contrib/vagrant docbug find
* @ecordell - `BUILDPACK_URL` suggestion, 204 logs fix
* @edude03 - platform logging help
* @gregwebs - doc reviewing and feedback
* @hyao - NFS for Ubuntu doc fix
* @itsmeduncan - CoreOS feedback, FLEETCTL_TUNNEL fix, 3-node cluster discussion
* @jacobwg - registration_enabled bug find
* @jefff - NFS / OS X issue report
* @johanneswuerbach - toggle registration in etcd, configurable gzip settings
* @jperville - deploy / scale issue, caching discussion, fleet-check Makefile fix
* @justicel - Ruby buildpack testing
* @marceldegraf - `deis scale` 0.8.0 bug report
* @mattgreen - Buildpack / dokku info
* @nathansamson - custom domains support, testing and HA discussion
* @olarcheveque - doc review, deis.io/get-deis bug catch
* @possibilities - NFS / OS X issue report
* @recipher - 3-node Vagrant testing, HA discussion
* @rgarcia - testing, db migration suggestion, VPC instructions
* @robparrott - deprecate NFS & update CoreOS PR
* @romansergey - registration_enabled bug find
* @SEJeff - vulcand discussion
* @shanejonas - Docker container template timeout fix
* @siliconcow - App.destroy() 500 error report, logger bug find
* @stefanfoulis - testing, Referer header fix, deis logger bug find
* @themasterchef - NFS / OS X password, vagrant halt + restart issues
* @tricky42 - registration_enabled bug find
* @viglesiasce - JSON prettyprint PR
* @wedtm - NodeJS and CLI binary testing, bug finding
* @wenxian - testing, fleetctl bug catch
* @zachlatta - 'clusters:create' reviewing

## Known Issues

### Container Timeouts
We are still seeing stability issues with the public Docker Index and its
content delivery network. When bootstrapping a Deis cluster, this can manifest
as timeout errors, missing images and/or missing layers.  Manual `systemctl`
commands will usually resolve the issue.  We are working with the Docker team on
the root issue.

## What's Next?

### Deis Upgrades

In our push toward 1.0, our most important priority is making sure Deis upgrades
are fast, painless and backwards compatible. We are devoting significant
resources to this effort, and are working closely with the CoreOS team who has a
great deal of experience with rolling upgrades.

### Improved Docker Integration

We have heard from the community and our customers that Deis needs a way to
deploy and promote existing Docker images outside of a `git push` style build
process. This is critical for implementing CI/CD pipelines on top of Deis. We
are currently investigating the best workflow in conjunction with our friends at
Docker HQ. We hope this new feature, combined with enhanced support for pushing
`Dockerfile` repositories, will provide the best Docker-native experience of any
PaaS on the market.

### Platform Security & HA

With Deis being used for more and more real-world deployments, we need to
address remaining
[security](https://github.com/deis/deis/issues?labels=security&state=open)
issues as well as
[high-availability](https://github.com/deis/deis/issues/984) of the platform
itself. Deis components and hosted apps are now fronted by a user-defined number
of routers, and other HA features will be a prerequisite for our 1.0 release.

## Future

### Service Gateway
We need to make it as easy as possible for ops folks to publish a set of
reusable backing services (databases, queues, storage, etc) and allow developers
to attach those services to applications. This will be done in a loosely coupled
way, following Twelve Factor best practices. You can review the initial
implementation and follow progress
[on this GitHub issue](https://github.com/opdemand/deis/issues/231).

### Interactive `deis run`
Though we provide the ability to run admin commands inside containers, we don't
currently support interactive shells into containers (i.e. `deis run bash`).
Once this infrastructure is in place, this will also allow us to implement log
tailing and other real-time features.

## How can you help?

* Star our [GitHub repository](https://github.com/opdemand/deis)
* Help spread the word about [@opendeis](http://twitter.com/opendeis)
  on Twitter
* Join the conversation in the [#deis channel](https://botbot.me/freenode/deis/)
  on Freenode
* Pick an [easy-fix issue](https://github.com/deis/deis/issues?labels=easy-fix&state=open)
  and start contributing!

Learn about other ways to [get involved](http://deis.io/get-involved/) on
our website.
