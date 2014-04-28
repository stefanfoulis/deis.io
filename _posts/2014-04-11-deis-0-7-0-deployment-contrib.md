---
layout: post
title: Deis 0.7.0 - Deployment Resiliency, Contrib Guidelines
author: mboersma
meta:
  - name: description
    content: This post explains our research on different reverse proxies.
  - name: keywords
    content: deis, release, 0.7.0
excerpt: The Deis project is happy to announce our v0.7.0 release. This version improves quality and testing for our Chef recipes, allows a Deis admin to disable user registrations, and adds the `deis shortcuts` command. Deis also has new guidelines to help community members contribute.
---

The Deis project is happy to announce our v0.7.0 release. This version improves quality and testing for our Chef recipes, allows a Deis admin to disable user registrations, and adds the `deis shortcuts` command. Deis also has new [guidelines][1] to help community members contribute.

<!--more-->

## What is Deis?

Deis is a Django/Celery API server, Python CLI and set of [Chef cookbooks][2] that combine to provide a Heroku-inspired application platform for public and private clouds. Your PaaS. Your Rules.

## 0.7.0 Summary

### New Features

*   `deis shortcuts` shows how to type less when using the Deis CLI
*   `REGISTRATION_ENABLED = False` allows a Deis admin to disallow new user registrations
*   [CONTRIBUTING.md][1] clarifies expectations for code contributed by Deis community members
*   Added a default nginx response for proxy nodes
*   Added a friendly name in Vagrant for the deis-controller VM

### Under the Hood

*   Added Chefspec tests, serverspec tests, foodcritic linting, and TestKitchen automation to deis-cookbook
*   Added NTP client to server recipes
*   Cleaned up cookbook structure
*   Added `--force-yes` to install some required .deb packages
*   Install `lxc` package explicitly on Ubuntu
*   Updated Docker to 0.9.0

### Bug Fixes

*   Vagrant contrib script creates Chef data bags again
*   Pinned pep8, pyflakes, and flake8 to fix CI test errors
*   Use `from django.conf import settings`, not `from deis...`
*   Updated docs to catch up with the new [OpsCode][3] portal
*   Fixed a DigitalOcean API call updated in dop 1.6
*   Fixed branching logic for proxies in Chef recipe
*   Removed `docker pull` timeouts
*   Removed unused source attributes in `default.rb`
*   Better codeblock spacing in docs

### Community Shout-Outs

We want to thank the following Deis community members for creating GitHub issues, providing support to others, and working on various Deis branches:

*   @alexanderkiel - lots of testing, images aren't deleted issue
*   @benton - found issue when a project contains both a `Procfile` and a `Dockerfile`
*   @bflad - lots of chef and chef-docker help, ChefSpec Runners PR
*   @dginther - testing, 500 errors hide EC2 response code issue
*   @foxycoder - add NTP client to packages
*   @johanneswuerbach - recommended EC2 m3 instances, faster + cheaper
*   @jstop - testing and `--force-yes` fixes
*   @zachlatta - testing, request for database example docs, Rackspace instance scaling issue

## Known Issues

### `deis logs` fails in some environments

The CLI command to display application logs may fail with a 403 error from rsyslog. We think this may occur when proper reverse DNS name resolution is unavailable. We are working on a solution for the *deis/logger* component.

### Proxy for Platform Services

As part of moving Deis into Docker containers, we had to change the exposed ports for some core platform services. For example, the Django API server is now exposed on 8000/tcp rather than 80/tcp. We will soon distribute a [new proxy service][4] that exposes the platform components on standard ports.

## What's Next?

### Enterprise-Grade Scheduler

We are now testing a new scheduler based on [Fleet][5] and [CoreOS][6]. This will dramatically simplify the Deis controller codebase and allow Deis clusters to scale to 10,000+ nodes.

### Remove the Chef Dependency

Though [Chef][7] will continue to be supported for deploying Deis, we are moving away from requiring a Chef Server and using Data Bags for cluster configuration.

### Promote artifacts from Docker Registry

In order to facilitate a streamlined CI/CD process, we need an ability to promote existing Docker images as builds (bypassing the `git push` process). We are currently investigating the best workflow.

### SSL & General Security Improvements

For more details, [see issues tagged security][8] on GitHub.

## Future

### Service Gateway

We need to make it as easy for ops folks to publish a set of reusable backing services (databases, queues, storage, etc) and allow developers to attach those services to applications. This will be done in a loosely coupled way, following Twelve Factor best practices. You can review the initial implementation and follow progress [on this GitHub issue][9].

### Interactive `deis run`

Though we provide the ability to run admin commands inside containers, we don't currently support interactive shells into containers (i.e. `deis run bash`). Once this infrastructure is in place, this will also allow us to implement log tailing and other real-time features.

## How can you help?

*   Star our [GitHub repository][10]
*   Help spread the word about [@opendeis][11] on Twitter
*   Explore contributing to the Deis project by joining the #deis channel on Freenode

You can learn about other ways to [get involved][12] on our website.

 [1]: https://github.com/opdemand/deis/blob/master/CONTRIBUTING.md
 [2]: https://github.com/opdemand/deis-cookbook
 [3]: https://manage.opscode.com/
 [4]: https://github.com/opdemand/deis/issues/535
 [5]: https://github.com/coreos/fleet
 [6]: https://coreos.com/
 [7]: http://www.getchef.com/chef/
 [8]: https://github.com/opdemand/deis/issues?labels=security&state=open
 [9]: https://github.com/opdemand/deis/issues/231
 [10]: https://github.com/opdemand/deis
 [11]: http://twitter.com/opendeis
 [12]: http://deis.io/get-involved/