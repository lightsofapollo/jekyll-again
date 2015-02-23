---
layout: post
title: "Gaia + Taskcluster + Treeherder"
description: "Next generation CI for gaia"
category: mozilla, ci, gaia
tags: [ci, docker, mozilla, gaia]
---
# What is this stuff?

(originally posted on dev-gaia)

For some time now Gaia developers have wanted the ability to scale their tests infinitely, while reporting to a dashboard that both sheriffs and devs can monitor, and yet still maintain control over the test configurations themselves.

Taskcluster & Treeherder let's us do this: http://treeherder-dev.allizom.org/ui/#/jobs?repo=gaia-master
Taskcluster http://docs.taskcluster.net/ drives the tests and with a small github hook allows us to configure the jobs from a json file in the tree (this will likely be a yaml file in the end) https://github.com/mozilla-b2g/gaia/blob/master/taskgraph.json

Treeherder is the next generation "TBPL" which allows us to report results to sheriffs from external resources (meaning we can control the tests) for both a "try" interface (like pull requests) and branch landings.

Crrently, we are very close to having green runs in treeherder, with only one intermittent and the rest green ...

# How is this different then gaia-try?

Taskcluster will eventually replace _all_ buildbot run jobs (starting with linux)... we are currently in the process of moving tests over and getting treeherder ready for production.

Gaia-try is run on top of buildbot and hooks into our github pull requests.. Gaia-try gives us a single set of suites that the sheriffs can look at and help keep our tree green. This should be considered "production".

Treeherder/taskcluster are designed to solve the issues with the current
buildbot/tbpl implementations:

  - in tree configuration

  - complete control over the test environment with docker (meaning you can have the exact same setup locally as on TBPL!)

  - artifacts for pull requests (think screenshots for failed tests,
    gaia profiles, etc...)

 - in tree graph capabilities (for example "smoketests" builds by running smaller test
   suites or how tests depend on builds).

# How is this different from travis-ci?

 - we can scale on demand on any AWS hardware we like (at very low cost thanks
   to spot)

 - docker is used to provide a consistent test environment that may be run
   locally

  - artifacts for pull requests (think screenshots for failed tests,
    gaia profiles, etc...)

 - logs can be any size (but still mostly "live")

 - reports to TBPL2 (treeherder)

# When is this production ready?

taskcluster + treeherder is _not_ ready for production yet... while the tests are running this is not in a state where sheriffs can manage it (yet!). Our plan is to continue to add taskcluster test suites (and builds!) for all trees (yes gecko) and have them run in parallel with the buildbot jobs this month...

I will be posting weekly updates on my blog about taskcluster/treeherder http://lightsofapollo.github.io/ and how it effects gaia (and hopefully your overall happiness)

# Where are the docs??

  - http://docs.taskcluster.net/
  - (More coming to gaia-taskcluster and gaia readme as we get closer to
    production)

# WHERE IS THE CODE?

 - https://github.com/taskcluster (overall project)
 - https://github.com/lightsofapollo/gaia-taskcluster (my current gaia intergration)
 - https://github.com/mozilla/treeherder-service (treeherder backend)
 - https://github.com/mozilla/treeherder-ui (treeherder frontend)
