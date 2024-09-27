---
layout: page
title: Pykka
parent: Actors
description: Using Pykka to build an Actor-Based Model in python
nav_order: 1
tags: [python, pykka, actors, async]
---

# Actor-Based Model - Pykka
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


**Pykka** is a Python library for building actor-based concurrent applications. It simplifies writing concurrent and asynchronous code by introducing actors—isolated units of computation that communicate through messages. This guide will demonstrate how to use Pykka to build an actor-based data science model, allowing for scalable, asynchronous data processing.

---

### 1. Why Use Actors in Data Science?

In data science, especially when processing large datasets or performing complex computations, we often need to deal with parallelism and concurrency. Common use cases for actor models in data science include:

- **Asynchronous I/O**: Reading and processing data from files, databases, or APIs without blocking the main program.
- **Task Parallelism**: Distributing heavy tasks (e.g., training models, processing chunks of data) across multiple cores.
- **Fault Tolerance**: Isolating parts of the system so that failures in one component don’t crash the entire system.
- **Scalability**: Scaling up computations by distributing them across multiple actors that can run concurrently.

The actor model fits well into these scenarios by ensuring modular, non-blocking, and fault-tolerant code.

---

### 2. Introduction to Pykka

Pykka is a Python library that brings the actor model to Python, enabling you to write concurrent and asynchronous programs. Each actor in Pykka is an independent worker that can send and receive messages, run tasks asynchronously, and handle failures separately.

#### Key Concepts:
- **Actor**: An object that performs tasks in isolation. It has a mailbox for receiving messages.
- **Message**: The communication unit between actors.
- **Future**: An object representing a result that may not yet be available. You can get the result once it's computed.

---

### 3. Setting Up Pykka

To install Pykka, simply run:

```bash
pip install pykka
```

Once installed, you're ready to start using Pykka to build an actor-based system.

---

### 4. Building an Actor-Based Data Science Model

In this section, we’ll build a data processing pipeline using Pykka where each stage of the pipeline is handled by a separate actor.

#### Example Use Case:
We'll create a pipeline to load data, preprocess it, and then apply a machine learning model asynchronously.

#### 4.1. Defining Actor Classes

We’ll define individual actors for each stage in our pipeline. Actors will handle tasks like loading data, preprocessing, and applying models.

```python
import pykka
import pandas as pd

# Actor for loading data
class DataLoaderActor(pykka.ThreadingActor):
    def load_data(self, filepath):
        print(f"Loading data from {filepath}")
        data = pd.read_csv(filepath)  # Load CSV data
        return data

# Actor for preprocessing data
class DataPreprocessorActor(pykka.ThreadingActor):
    def preprocess(self, data):
        print("Preprocessing data...")
        # Basic preprocessing steps
        data = data.dropna()  # Remove missing values
        data = (data - data.mean()) / data.std()  # Standardize data
        return data

# Actor for applying a model
class ModelActor(pykka.ThreadingActor):
    def apply_model(self, data):
        print("Applying model...")
        # Dummy model prediction for demonstration
        predictions = data.sum(axis=1)  # Placeholder for model
        return predictions
```

#### 4.2. Creating Message-Driven Actors

Each actor can receive messages and act on them asynchronously. Let's create an entry point where actors are instantiated, and messages are sent between them.

```python
if __name__ == '__main__':
    # Start actors
    data_loader = DataLoaderActor.start()
    preprocessor = DataPreprocessorActor.start()
    model_actor = ModelActor.start()

    # Load data asynchronously
    future_data = data_loader.ask({'method': 'load_data', 'args': ('data.csv',)})
    
    # Chain preprocessing and modeling asynchronously
    future_data.add_callback(
        lambda data: preprocessor.ask({'method': 'preprocess', 'args': (data,)})
        .add_callback(lambda preprocessed_data: model_actor.ask({'method': 'apply_model', 'args': (preprocessed_data,)}))
    )

    # Wait for results
    predictions = future_data.get(timeout=10)  # Wait up to 10 seconds
    print(predictions)
```

#### 4.3. Connecting Actors for a Data Pipeline

Each actor is responsible for one stage in the pipeline. We use the `ask()` method to send messages, and the result is returned as a future. We can chain actors by passing the result of one actor as input to another.

In the above example:
1. The `DataLoaderActor` loads data.
2. The `DataPreprocessorActor` preprocesses the data.
3. The `ModelActor` makes predictions.

---

### 5. Handling Results Asynchronously

In Pykka, results of actor computations are returned as `Future` objects. You can use `.get()` to retrieve the result when it’s ready. If the result depends on multiple asynchronous computations, you can chain them using callbacks:

```python
future_data.add_callback(lambda data: print(f"Data loaded: {data.shape}"))
```

Alternatively, you can use `.then()` to chain future results.

---

### 6. Error Handling and Actor Lifecycles

Actors may fail due to unexpected errors. Pykka provides ways to handle errors gracefully:

- **Error Propagation**: If an actor fails, the future object associated with the failed task will hold an exception.
  
  ```python
  try:
      result = future_data.get()
  except pykka.ActorDeadError:
      print("Actor failed to process the request.")
  ```

- **Stopping Actors**: It’s important to stop actors when their job is done.

  ```python
  data_loader.stop()
  preprocessor.stop()
  model_actor.stop()
  ```

---

### 7. Testing and Debugging

Testing Pykka actors can be done using standard testing frameworks like `unittest` or `pytest`. Pykka provides useful tools to test asynchronous behavior:

```python
def test_data_loading():
    loader = DataLoaderActor.start()
    data = loader.ask({'method': 'load_data', 'args': ('test_data.csv',)}).get(timeout=5)
    assert data is not None
    loader.stop()
```

Debugging actor behavior often involves tracing messages and actor states. Pykka allows you to log actor events and messages, which is useful for diagnosing issues in the pipeline.

---

### 8. Conclusion

Using Pykka to build an actor-based asynchronous data science model offers several advantages, including modularity, fault tolerance, and scalability. By organizing your data pipeline into separate, isolated actors, you can handle complex computations more effectively, manage failures better, and build systems that are easier to scale and maintain.

In summary:
- **Actors** encapsulate each stage of the data processing pipeline.
- **Messages** are used to communicate between actors asynchronously.
- **Future** objects allow you to manage asynchronous results and chain computations.
  
This approach is ideal for large-scale, parallelisable data science workflows where tasks can run concurrently.