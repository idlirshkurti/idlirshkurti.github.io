---
layout: page
title: Ray Actors
parent: Actors
description: Distributed Actor-Based Models with Ray in Python
nav_order: 3
tags: [python, ray, actors, distributed, async]
---

# Scaling Concurrent Data Science with Ray Actors
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

While **Pykka** and **Asyncio** provide excellent concurrency on a single machine, **Ray** is the standard for scaling actor-based models across a distributed cluster. This guide explores Ray Actors, how they work, and how they compare to the local-first approaches of Pykka and Asyncio.

### What are Ray Actors?

In Ray, an **Actor** is a stateful worker. When you instantiate a class as a Ray Actor, Ray creates a "service" on a worker process (potentially on a different machine in your cluster). This worker maintains its own internal state, which is preserved across multiple remote method calls.

#### Key Features:
- **Distributed by Default**: Actors can run on any node in a Ray cluster.
- **State Preservation**: Unlike Ray tasks (which are stateless), actors keep their internal state between calls.
- **Resource Management**: You can specify CPU/GPU requirements for each actor.

---

### Comparison: Ray vs. Pykka vs. Asyncio

| Feature | Asyncio Actors | Pykka Actors | Ray Actors |
| :--- | :--- | :--- | :--- |
| **Concurrency Model** | Single-threaded Event Loop | Threading (or Gevent) | Distributed Processes |
| **Scaling** | Vertical (Limited to 1 CPU) | Vertical (Multiple Threads/Cores) | Horizontal (Multi-node Cluster) |
| **Communication** | `asyncio.Queue` / Direct await | `ask` / `tell` with Futures | `.remote()` method calls |
| **Isolation** | Lowest (Shared memory) | Medium (Thread-safe) | Highest (Isolated Processes) |
| **Complexity** | Simple (Built-in) | Moderate (Library needed) | High (Infrastructure required) |

#### Similarities:
- All three follow the **Actor Model** core principles: isolation, statefulness, and asynchronous messaging.
- They all use some form of "Future" or "Object Reference" to handle results that haven't been computed yet.
- They help avoid race conditions by ensuring only one message is processed at a time per actor.

---

### Implementation Example: A Distributed Counter

Here is how you define and use a Ray Actor, compared to the concepts we've seen in Pykka and Asyncio.

#### 1. Defining the Actor

```python
import ray

# Initialize Ray (local or cluster)
ray.init()

@ray.remote
class CounterActor:
    def __init__(self):
        self.value = 0

    def increment(self):
        self.value += 1
        return self.value

    def get_value(self):
        return self.value
```

#### 2. Interacting with the Actor

In Ray, you use `.remote()` for both instantiation and method calls. This is similar to Pykka's `.start()` and `.ask()`, or Asyncio's queue-based `send()`.

```python
# Instantiate the actor
counter = CounterActor.remote()

# Call methods asynchronously
# These return "ObjectRefs" (similar to Pykka Futures)
future1 = counter.increment.remote()
future2 = counter.increment.remote()

# Retrieve results (blocking call)
print(ray.get([future1, future2]))  # Output: [1, 2]
```

---

### Why use Ray for Data Science?

In a data science pipeline, you might use different actor frameworks depending on the scale:

1. **Asyncio Actors**: Best for I/O-bound tasks like fetching data from 100 different APIs on a single machine.
2. **Pykka Actors**: Best for CPU-bound tasks that need to run in parallel on a single machine (e.g., local model inference).
3. **Ray Actors**: Essential when your model is too large for one GPU, or when you need to process terabytes of data across 50 machines.

#### Use Case: Distributed Model Serving

Imagine a large Language Model (LLM) that requires a full GPU. You can spin up multiple Ray Actors, each holding a copy of the model on a different GPU/node, and load balance requests across them.

```python
@ray.remote(num_gpus=1)
class ModelServer:
    def __init__(self, model_id):
        self.model = load_model(model_id)
        
    def predict(self, data):
        return self.model.generate(data)

# Scale to 4 GPUs
servers = [ModelServer.remote("llama-3") for _ in range(4)]
```

---

### Conclusion

Ray Actors take the concept of stateful, asynchronous computation and scale it to the cloud. While **Asyncio** is your tool for local speed and **Pykka** for local structure, **Ray** is the engine for massive distribution. Understanding how to bridge these models allows you to build data systems that are as responsive as they are scalable.

---

### References
- [Ray Documentation: Actors](https://docs.ray.io/en/latest/ray-core/actors.html)
- [Building Asyncio Actors (Internal Tutorial)](./asyncio_actors.md)
- [Building Pykka Actors (Internal Tutorial)](./pykka_actors.md)
