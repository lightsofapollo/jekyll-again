---
layout: post
title: "Taskcluster - Mozilla's new test infrastructure project"
description: "wtf is a taskcluster"
category: mozilla, ci
tags: [ci, docker, mozilla]
---

[Taskcluster](https://github.com/taskcluster) is not one singular entity that runs a script with output in a pretty interface or a github hook
listener, but rather a set of decoupled interfaces that enables us to
build various test infrastructures while optimizing for cost, performance and reliability. 
The focus of this post is Linux. I will have more information how this works for OSX/Window soon.

## Some History

Mozilla has [quite](http://dxr.mozilla.org/mozilla-central/source/)
[a](https://github.com/mozilla) [few](https://github.com/mozilla-b2g) different
code bases, most depend on [gecko](https://github.com/mozilla/gecko-dev)
(the heart of Firefox and FirefoxOS). 
Getting your project hooked up to our [current CI
infrastructure](https://tbpl.mozilla.org)
usually requires a multi-team process that takes days or more.
Historically, simply merging projects into gecko was easier than having
external repositories that depend on gecko, which our current CI cannot
easily support. 

It is critical to be able to see in one place ([TBPL](https://tbpl.mozilla.org))
that all the projects depend on gecko are working.
Today TBPL current this process is tightly coupled to our buildbot infrastructure (which together
make up our current CI). If you really care about your project 
not breaking when a change lands in gecko, you really only have one option:
hosting your testing infrastructure under buildbot (which feeds TBPL).

## Where Taskcluster comes in

[Treeherder](https://wiki.mozilla.org/Auto-tools/Projects/Treeherder)
resolves the tight coupling problem by separating the reporting from the
test running process. This enables us to re-imagine our workflow and how it's
optimized. We can run tests anywhere using any kind of utility/library
assuming it gives us the proper hooks (really just logs and some
revision information) to plug results into our development workflow.

A high level workflow with taskcluster looks like this:

You submit some code (this can be patch or a pull request, etc...) to a
"scheduler" ( [I have started on one for gaia](https://github.com/lightsofapollo/gaia-taskcluster) )
which submits a set of tasks. Each task is run inside a [docker](https://www.docker.io/) container
the container's image is specified as part of your task. This means anything you can 
imagine running on linux you can directly specify in your container
(no more waiting for vm reimaging, etc...) this also means we
directly control the resources that container uses (less variance in
test) AND if something goes wrong you can download the entire
environment that test ran on locally to debug it.

As tasks are completed the [task cluster queue](https://github.com/taskcluster/taskcluster-queue)
emits events over AMQP (think pulse) so anyone interested in the status
of tests, etc.. can hook directly into this... This enables us to post
results as they happen directly to treeherder.

The initial taskcluster [provisions](https://github.com/taskcluster/aws-provisioner)
AWS spot nodes on demand (we have it capped to a fixed number right now) so during peaks
we can burst to an almost unlimited number of nodes. During idle times
workers shut themselves down to reduce costs. We have additional plans
for different clouds (and physical hardware on open stack).

Each component can be easily replaced (and multiple types of workers and
provisioners can be added on demand. [Jonas Finnemann Jensen](http://jonasfj.dk/blog/) has done a awesome job
documenting how taskcluster works [in the docs](http://docs.taskcluster.net/queue/)
at the API level.

## What the future looks like

My initial plan is to hook everything up for [gaia](https://github.com/mozilla-b2g/gaia)
the FirefoxOS frontend. This will replace our current travis CI setup.

As pull requests come in we will run tests on taskcluster and report
status to both treeherder and github (the beloved [github status api](https://github.com/blog/1227-commit-status-api)).
The ability to hook up new types of tests from the tree itself (and test
new types from the tree itself) will continue on in the form of a task
template (another blog post coming). Developers can see the status of
their tests from treeherder.

Code landing in master follows the same practice and results will report
into a gaia specific treeherder view.

Most importantly immediately after treeherder is launched we can run all
gaia testing on _the same exact infrastructure for both gaia and gecko
commits_ [Jonas Sicking (b2g overload)](https://twitter.com/SickingJ)
has some great ideas about locking gecko <-> gaia versions to reduce
another kind of failure which occurs when developing against the ever
changing landscape of gecko / gaia commits.

When is the future? We have implemented the "core" of taskcluster
already and have the ability to run tests. By the end of the month
(March) we will have the capability to replace the entire gaia workflow with
taskcluster.

## Why not X CI solution

Building a brand new CI solution is non-trivial why are we doing this?

  - To leverage LXC containers (docker): One of the big problems we hit
    when trying to debug test failures is the vairance of testing
    locally and remotely. With LXC containers you can download the
    entire container (the entire environment which your test runs in)
    and run it with the same cpu/memory/swap/filesystem as it would
    run remotely.

  - On demand scaling. We have (somewhat predictable) bursts throughout 
    the day and the ability to spin up (and down) on demand is required to 
    keep up with our changing needs throughout the day.

  - Make in tree configuration easy. Pull requests + in tree configuration
    enable developers to quickly iterate on tests and testing infrastructure

  - Modular extensible components with public facing APIs. Want run
    tasks to do things other then test/build or report to something other
    then treeherder? [We have or will build an api for that](http://docs.taskcluster.net).
    
    Hackability is imporant... The parts you don't want to solve
    (running aws nodes, keeping them up, pricing them, etc...) are
    solved for you so you can focus on building the next great mozilla
    related thing (better bisection tools, etc...).

  - More flexibility to test/deploy optimizations... We have something
    like a compute year of tests and 10-30+ minute chunks of testing is
    normal. We need to iterate on our test infrastructure quickly to try
    to reduce this where possible with CI changes.

Here are a few potential alternatives below... I list out the pros &
cons of each from my perspective (and a short description of each).

### Travis [hosted]

TravisCI is an awesome [free] open source testing service that we use
for many of our smaller projects. 
  
Travis works really well for the 90% webdev usecase. 
Gaia does not fit well into that use case and gecko does so even less.

Pros:

  - Dead simple setup.
  - Iterate on test frameworks, etc... on every pull request without any
    issue.
  - Nice simple UI which reports live logging.
  - Adding tests and configuring tests is trivial.

Cons:

  - Difficult to debug failures locally.
  - No public facing API for creating jobs.
  - No build artifacts on pull requests.
  - Cannot store arbitrarily long logs (this is only an issue for open source IIRC).
  - On demand scaling.

### Buildbot [build on top of it]

We currently use buildbot at scale thousands~ of machines for all gecko
testing on multiple platforms. If you are using firefox it was built by
our buildbot setup.

(NOTE: This is a critique of how we currently use buildbot not the
entire project). If I am missing something or you think a CI solution
could fit the bill contact me!

Pros:

  - We have it working at a large scale already.

Cons:

  - Adding tests and configuring tests is fairly difficult and involves
    long lead times.
  - Difficult to debug failures locally.
  - Configuration files live outside of the tree.
  - Persistent connection master/slave model.
  - Its one monolithic project which is difficult to replace components
    of.
  - Slow rollout of new machine requirements & configurations.

### Jenkins

We are using Jenkins for our on device testing.

Pros:

  - Easy to configure jobs from the UI (decent ability to do
    configuration yourself).
  - Configuration (by default) does not live in the tree.
  - Tons of plugins (with varying quality).

Cons:

  - By default difficult to debug failures locally.
  - Persistent connection master/slave model.
  - Configuration files live outside of the tree.

### Drone.io [hosted/not hosted]

[Drone.io](https://github.com/drone/drone) recently open sourced... It's
docker based and shows promise. Out of all the options above it looks the 
closest to the to what we want for linux testing.

I am going to omit the Pros/Cons here the basics look good for drone and
it requires some more investigation. Some missing things here
are:

  - A long term plan for supporting multiple operating systems. 
  - A public api for scheduling tasks/jobs.
  - On demand scaling.

