---
layout: post
title: "How to break a Monolith into Microservices"
date: 2021-09-09 22:00:00 +0800
tags: Server DesignPattern
---

referance: [How to break a Monolith into Microservices](https://martinfowler.com/articles/break-monolith-into-microservices.html)

- target:

  - increasing the scale of operation
  - accelerating the pace of change(avoid high cost)
  - grow the number of teams
  - experiment and deliver value faster

- Prerequisites

  - pipelines to independently build, test, and deploy
  - secure
  - debug
  - monitor

- The Microservice Ecosystem Destination

  - Each microservice expose an API that developers can discover and use in a self-serve manner.
  - Microservices have independent lifecycle(build, test, release).
  - The microservices ecosystem enforces an organizational structure of autonomous long standing teams, each responsible for one or multiple services.

- couple things: data, logic, APIs

- principle
  - start from edge services first
  - Minimize Dependency Back to the Monolith
    - In cases where a new service ends up with a call back to the monolith, I suggest to expose a new API from the monolith, and access the API through an anti-corruption layer in the new service to make sure that the monolith concepts do not leak out.
  - Split Sticky Capabilities Early
  - Decouple Vertically and Release the Data Early
  - Decouple What is Important to the Business and (historically)Changes Frequently
  - Decouple Capability and not Code
  - Go Macro First, then Micro
  - Migrate in Atomic Evolutionary Steps
