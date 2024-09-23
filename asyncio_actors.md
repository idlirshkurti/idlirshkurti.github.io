---
layout: page
title: Asyncio actors
parent: Actors
description: Using asyncio to build an Actor-Based Model manually
nav_order: 1
tags: [python, asyncio, actors, concurrency, async]
---

# Building Actor-Based Asynchronous Data Science Models Manually with Python's `asyncio`
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


This guide walks you through creating an actor-based system for asynchronous data processing using Python’s `asyncio`. Instead of using Pykka, we'll build a custom `AsyncActor` class to handle asynchronous messages and data processing.

### Why Build Your Own Actor Framework?

Creating a custom actor system provides flexibility to:
- Design actors tailored to your specific data science workflow.
- Handle messages and tasks asynchronously without third-party dependencies.
- Gain full control over concurrency, error handling, and scalability.

---

### 1. Setting Up the Custom Actor System

We begin by creating a base class `AsyncActor` that manages the message queue and defines how actors send and receive messages asynchronously.

```python
import asyncio
import logging
from typing import Any, Optional

# Initialize the logger for this module
_logger = logging.getLogger(__name__)

class AsyncActor:
    """Base class for an asynchronous actor."""

    def __init__(self) -> None:
        """Initialize the actor with an async message queue."""
        self.queue: asyncio.Queue[Any] = asyncio.Queue()

    async def send(self, message: Any) -> None:
        """Send a message to the actor.
        
        Args:
            message: The message to send to the actor.
        """
        _logger.info(f"Sending message: {message}")
        await self.queue.put(message)

    async def receive(self) -> Any:
        """Receive a message from the actor's queue.

        Returns:
            The message received from the actor's queue.
        """
        return await self.queue.get()

    async def run(self) -> None:
        """Run the actor, continuously processing incoming messages."""
        while True:
            message = await self.receive()
            _logger.info(f"Received message: {message}")
            await self.on_receive(message)

    async def on_receive(self, message: Any) -> Optional[Any]:
        """Handle incoming messages, to be implemented by subclasses.

        Args:
            message: The incoming message to process.
        
        Raises:
            NotImplementedError: If the method is not implemented by the subclass.
        """
        raise NotImplementedError("on_receive method must be implemented by the subclass")
```

### 2. Building Actor Subclasses for Data Processing

To handle specific tasks in a data science pipeline (e.g., data loading, preprocessing, and applying models), we create actor subclasses that implement the `on_receive` method.

#### Data Loader Actor

```python
class DataLoaderActor(AsyncActor):
    """Actor responsible for loading data asynchronously."""
    
    async def on_receive(self, message: str) -> Optional[Any]:
        _logger.info(f"Loading data from: {message}")
        # Simulate data loading
        await asyncio.sleep(1)  # Simulate I/O-bound task
        data = f"Data loaded from {message}"
        _logger.info(f"Data loaded: {data}")
        return data
```

#### Preprocessor Actor

```python
class DataPreprocessorActor(AsyncActor):
    """Actor responsible for preprocessing data."""
    
    async def on_receive(self, message: Any) -> Optional[Any]:
        _logger.info(f"Preprocessing data: {message}")
        # Simulate data preprocessing
        await asyncio.sleep(1)  # Simulate CPU-bound task
        preprocessed_data = f"Preprocessed {message}"
        _logger.info(f"Data preprocessed: {preprocessed_data}")
        return preprocessed_data
```

#### Model Actor

```python
class ModelActor(AsyncActor):
    """Actor responsible for applying a machine learning model."""
    
    async def on_receive(self, message: Any) -> Optional[Any]:
        _logger.info(f"Applying model to: {message}")
        # Simulate model application
        await asyncio.sleep(1)  # Simulate compute-bound task
        result = f"Prediction based on {message}"
        _logger.info(f"Model prediction: {result}")
        return result
```

### 3. Connecting Actors in a Pipeline

Now that we have the actors, we can connect them to form a data pipeline. We’ll instantiate the actors and send messages through them to process data in stages.

```python
async def main():
    # Create actor instances
    data_loader = DataLoaderActor()
    preprocessor = DataPreprocessorActor()
    model_actor = ModelActor()
    
    # Start actor tasks (running in the background)
    asyncio.create_task(data_loader.run())
    asyncio.create_task(preprocessor.run())
    asyncio.create_task(model_actor.run())

    # Load data
    await data_loader.send('data.csv')
    raw_data = await data_loader.receive()
    
    # Preprocess data
    await preprocessor.send(raw_data)
    preprocessed_data = await preprocessor.receive()
    
    # Apply model
    await model_actor.send(preprocessed_data)
    prediction = await model_actor.receive()

    _logger.info(f"Final prediction: {prediction}")

# Running the pipeline
if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    asyncio.run(main())
```

### 4. Handling Results and Asynchrony

Each actor operates independently and processes messages asynchronously, meaning tasks like data loading and model inference can run without blocking each other. The `asyncio.Queue` ensures that messages are processed one at a time, preventing race conditions.

You can also chain results between actors. For example, after loading data, you can send the result to the preprocessor and then forward the preprocessed data to the model.

### 5. Error Handling and Graceful Shutdown

It’s important to handle errors and ensure that actors can shut down gracefully when their work is done.

#### Handling Errors

```python
class SafeActor(AsyncActor):
    async def run(self) -> None:
        try:
            await super().run()
        except Exception as e:
            _logger.error(f"Actor failed: {e}")
```

#### Graceful Shutdown

You can stop an actor by signaling it to exit the message loop.

```python
class StoppableActor(AsyncActor):
    async def run(self) -> None:
        running = True
        while running:
            message = await self.receive()
            if message == 'stop':
                running = False
            else:
                await self.on_receive(message)

# In main function:
await actor.send('stop')  # This stops the actor
```

### 6. Testing and Debugging

When building an asynchronous actor system, testing becomes more challenging because of the concurrent nature of tasks. Use `unittest` or `pytest` with `asyncio` support.

#### Example Test for DataLoaderActor

```python
import unittest
import asyncio

class TestAsyncActors(unittest.IsolatedAsyncioTestCase):
    async def test_data_loader(self):
        loader = DataLoaderActor()
        asyncio.create_task(loader.run())
        await loader.send('test_data.csv')
        result = await loader.receive()
        self.assertEqual(result, "Data loaded from test_data.csv")

if __name__ == '__main__':
    unittest.main()
```

---

### Conclusion

By building a custom actor framework using Python's `asyncio`, you gain full control over how actors interact, process messages, and handle concurrency. This approach allows you to build highly scalable and fault-tolerant data science pipelines. You can:
- Manage data asynchronously across different stages like loading, preprocessing, and model inference.
- Ensure isolation between tasks, making your system more robust and easier to maintain.
  
With this foundation, you can extend the framework to add more actors for tasks like feature engineering, model tuning, and real-time monitoring.