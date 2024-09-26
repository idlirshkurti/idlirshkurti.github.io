---
layout: page
title: Actors
parent: the-dev-hub
has_children: true
nav_order: 2
description: Building an Actors-Based Model in python
tags: [python, actors, async]
---

# Best practices and tools to build an Actor-Based Model in python

Notes and guidelines on building an Actor-Based model in python.


### Key Components of an Actor System

In an actor system:
1. **Actor**: Represents an independent unit of computation with its own state. Actors process messages one at a time asynchronously.
2. **Messages**: Actors communicate by sending messages to one another.
3. **Queue**: Messages are typically stored in an async queue until the actor is ready to process them.