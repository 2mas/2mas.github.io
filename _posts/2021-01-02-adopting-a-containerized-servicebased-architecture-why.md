---
title: "Adopting a containerized, servicebased architecture – Why?"
excerpt: "In this post I'll highlight some of my key takeaways when considering to adopt a containerized architecture."
date: 2021-01-02T08:00:30-04:00
categories:
  - blog
tags:
  - containers
  - microservices
---

###  Why?

There are different reasons behind making a decision to change a software architecture. It's not a trivial task to do, so there has to be legitimate reasons for it.

Look at what your current situation is and think about: are there any problems with this? Some common problems might be:

1. _Maintenance difficulties_ (huge topic, a small example follows below)
2. _Lack of scalability_
3. _Hard to test_
4. _Hard to host and develop_, requires specific components installed on a specific OS + versions. Requirements on the machine running the code might prevent you from using the best tool for the task, if that tool introduces new dependencies, SDK:s etc.

The first reason is probably very common, because systems grow. At first, there might be a small website with just a few pages and some simple logic behind it. It would be overkill to split everything up in different libraries and locations at this point.

Then feature requests start coming in, adding complexity, and without continously stopping, thinking and refactoring, it will end up as something very hard to maintain. A monolith will emerge, and code might end up in contexts where it doesn't belong if developers are not careful and has a good understanding of the entire application. The result in that case is a complex system that takes a long time to learn and where a small change in one area can have many unexpected side-effects, breaking other parts of the system.

This is very time consuming and difficult to estimate for new functionality, make changes to and to test.

#### How to fix this?
There are many books and resources about software architecture to address this, and one approach is to separate parts of a system into self-contained pieces with clear boundaries. In the coding world this is where libraries/projects get created that encapsulates related functionality.

For example in an E-commerce platform, it would be much easier to make a change to the shoppingcart if the code you are editing doesn't also contain logic for managing customers, products, payments, site-navigation or whatever.

#### Enter services
This is where a _service-oriented architecture_ (SOA) can help. The point of this architecture is to isolate areas of a system into their own services that run independently. This works very well with a containerized approach because if the shoppingcart-code gets packaged in its own container, it forces a clear way to interact with it by exposing endpoints. As long as those endpoints remains and responds correctly, you can't mess up any other parts of the system when making changes to just the shoppingcart.

It's called a _microservice-architecture_ when these services are very small and keeps their own data. The shoppingcart-service has only one job and it will be easier to maintain, test, monitor and scale if needed.

So, when planning for a change in the shoppingcart-area, you won't have to worry about possibly breaking the product-catalog. It also provides the option to scale just a single part of an application since every container runs independently.

#### Hosting and development benefits
Another important benefit is that a container works the same wherever it's being run. This means there are more options for hosting, and no more "works on my machine" requiring exact versions of OS, SDK:s, programs and tools locally installed on dev-machines and on servers in order to run. And if one service goes down, the entire system won't follow anymore, since they run in different processes on perhaps different hosts.

#### Generally
There are Linux and Windows containers and both can't be run on the same container runtime at the same time, so you would have to pick a path (you can have a Windows-container environment and another Linux-container environment working together with Kubernetes, but they would run on different servers/nodes).

I recommend that you choose Linux-containers unless you have specific reasons not to, like containerizing existing legacy applications without a rewrite or relying heavily on Windows-only services. This is because most technologies/runtimes/languages are being made to run in a Linux environment. The earlier exceptions of the .NET Framework is getting legacy since all paths of .NET going forward is as .NET core/5, that is a cross-platform runtime.

A service doesn't have to be micro in order to be beneficial, in fact going all-in micro-service might introduce more complex problems like handling distributed transactions for an example. Find the right solution for your system and your team of developers.

---

I'm planning a follow up about setting up a development environment with Docker and share some of my experiences on running containers with Kubernetes, if you're interested in that.
