---
layout: post
title: "Taskcluster Release Part 1 : Gecko"
description: "TC Update 1"
category: mozilla, ci
tags: [ci, docker, mozilla]
---

It's been awhile since my last blog post about taskcluster and I wanted to give an update...

## Taskcluster + Gecko

Taskcluster is running by default on

  - [try](https://treeherder.allizom.org/#/jobs?repo=try)
  - [b2g-inbound](https://treeherder.allizom.org/#/jobs?repo=b2g-inbound)
  - [mozilla-inbound](https://treeherder.allizom.org/#/jobs?repo=mozilla-inbound)
  - [fx-team](https://treeherder.allizom.org/#/jobs?repo=fx-team)
  - [mozilla-central](https://treeherder.allizom.org/#/jobs?repo=mozilla-central)


In Treeherder you will see jobs run by both buildbot and taskcluster. The "TC" jobs are
prefixed accordingly so you can tell the difference.

This is the last big step to enabling TC as the default CI for many mozilla
project. Adding new and existing branches is easily achieved with basic config changes.

Why is this a great thing? Just about everything is [in](https://dxr.mozilla.org/mozilla-central/source/testing/taskcluster/mach_commands.py) [the](https://dxr.mozilla.org/mozilla-central/source/testing/docker) [tree](https://dxr.mozilla.org/mozilla-central/source/testing/taskcluster/tasks/builds).

This means you can easily add new builds/tests and immediately push them
to try for testing (see the configs for [try](https://dxr.mozilla.org/mozilla-central/source/testing/taskcluster/tasks/branches/try/job_flags.yml)

Adding new tests and builds is easier than ever but the improvements don't stop there. Other key benefits on linux include:

#### We use [docker](https://dxr.mozilla.org/mozilla-central/source/testing/docker)

Docker enables easy cloning of CI environments.

```sh
# Pull tester image
docker pull quay.io/mozilla/tester:0.0.14
# Run tester image shell
docker run -it quay.io/mozilla/tester:0.0.14 /bin/bash
# <copy/paste stuff from task defintions into this>
```

#### Tests and builds are faster

Through this entire process we have been optimizing away overhead and using faster
machines which means both build (and particularly test) times are faster.

(Wins look [big](https://treeherder.allizom.org/#/jobs?repo=b2g-inbound&revision=233af1dfa476) but more in future blog post)

#### What's missing ?

 - Some tests fail due to differences in machines. When we move tests things
 fail largely due to timing issues (there are a few cases left here).

 - Retrigger/cancel does not work (yet!) as of the time of writing this it has
 not yet hit production but will be deployed soon.

 - Results currently show up only on [staging treeherder](https://treeherder.allizom.org).
   We will incrementally report these to production treeherder.
