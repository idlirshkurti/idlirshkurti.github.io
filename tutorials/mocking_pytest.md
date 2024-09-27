---
layout: page
title: Pytest mocking
parent: Pytest
description: Using pytest to mock async functions in python
nav_order: 1
tags: [python, pytest, mocking, async]
---

# Comprehensive Guide to Mocking in Pytest
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


Mocking is a crucial technique in software testing that allows you to replace real objects or functions with controlled ones. This technique is especially useful when you're testing isolated parts of your code and don't want external dependencies to interfere. In Python, pytest offers a flexible and powerful platform for testing, and mocking is one of the most commonly used features.
  

### Why Use Mocking?

When testing, especially in larger systems, you often have functions that rely on external services, databases, or I/O operations. Without mocks, your tests would:


* Be slow due to external service dependencies.

* Fail due to unavailability or unpredictability of external services.

* Be harder to test isolated parts of your codebase.


Mocking addresses these concerns by replacing the actual service calls, data, or objects with "mocked" versions that behave in a predictable way.

For example, you can mock a database call so that instead of querying a real database, your function gets a mock object that returns predefined data.
  

### Mocking Tools in Pytest


`unittest.mock`

The `unittest.mock` library, available in Python’s standard library, provides most of the mocking functionality used in pytest. The core components are:
  

`Mock()`: A flexible mock object that mimics any Python object.

`patch()`: A decorator/context manager that temporarily replaces real objects with mocks for testing.

`MagicMock()`: A subclass of `Mock()` with more advanced functionality, useful for mocking "magic methods" like __getitem__ or __call__.

### Pytest’s `monkeypatch` Fixture

In `pytest`, `monkeypatch` is a fixture that helps override or "monkey-patch" attributes or methods for testing. It’s very effective for temporarily replacing parts of the system you’re testing, like modules or functions, without making permanent changes.


#### Example: Using `monkeypatch`

```python

def get_username(user_id):
    # Imagine this function calls an external service to get a username
    raise Exception("External service not available!")

def test_get_username(monkeypatch):
    def mock_get_username(user_id):
        return "mocked_username"

    monkeypatch.setattr("module_containing_function.get_username", mock_get_username)
    
    result = get_username(1)
    assert result == "mocked_username"

```
  

In this example, monkeypatch replaces the real `get_username` function with a mock version that always returns "mocked_username" during the test. After the test, the original function is restored.

### How to Use `patch()`

The `patch()` function is one of the most versatile tools in mocking. It allows you to replace objects, functions, or methods during test execution. There are multiple ways to use patch(), including as a decorator, context manager, or manually.

#### Example: Mocking a Function with `patch()`

```python
from unittest.mock import patch

def fetch_data_from_api():
    # Simulate a real API call
    raise Exception("API not available")

@patch('module_containing_function.fetch_data_from_api')
def test_fetch_data(mock_fetch):
    mock_fetch.return_value = {"data": "mocked_data"}
    result = fetch_data_from_api()
    assert result == {"data": "mocked_data"}
```

In this example, the real `fetch_data_from_api` function is replaced by a mock object that returns predefined data. The test runs with this mocked version, allowing it to pass without calling the real API.


### Advanced Use Cases with MagicMock

`MagicMock()` extends `Mock()` by offering additional features for mocking magic methods, which are often needed in complex scenarios.


#### Example: Mocking a Class with `MagicMock`

```python
from unittest.mock import MagicMock

class Database:
    def connect(self):
        pass

def test_database_connection():
    mock_db = MagicMock(spec=Database)
    mock_db.connect.return_value = "Connection successful"
    
    assert mock_db.connect() == "Connection successful"
```

Here, we mock the `Database` class, and by using `MagicMock`, we define what happens when the `connect()` method is called. The spec argument ensures that the mock object follows the original class interface.

### Mocking Exceptions

Sometimes you want to simulate an error condition, like an external service throwing an exception. You can use mocks to simulate exceptions being raised when certain functions are called.

#### Example: Mocking an Exception

```python
from unittest.mock import patch

def risky_operation():
    raise Exception("Something went wrong")

@patch('module_containing_function.risky_operation')
def test_risky_operation(mock_risky):
    mock_risky.side_effect = Exception("Mocked exception")
    
    with pytest.raises(Exception, match="Mocked exception"):
        risky_operation()

```

In this example, the test ensures that the risky_operation function raises the mocked exception, allowing the test to verify that the error handling logic is correct.

### Mocking Asynchronous Functions

When dealing with asynchronous code, mocking becomes more complex because the mocked functions must also be asynchronous. Thankfully, `unittest.mock` has support for async mocks with `AsyncMock`.


#### Example: Mocking an Async Function

```python
from unittest.mock import AsyncMock

async def fetch_async_data():
    return "real data"

async def test_fetch_async_data():
    mock_fetch = AsyncMock(return_value="mocked data")
    
    result = await mock_fetch()
    assert result == "mocked data"
```

In this test, we create an async mock object that simulates the behavior of the real async function.
  

### Best Practices for Mocking

1. Mock Only What You Must

Over-mocking can lead to fragile tests. Always aim to mock external dependencies, not internal logic. This keeps your tests meaningful and reduces the risk of false positives.


2. Avoid Mocking Internal Libraries

Mocking internal libraries (like `os` or `json`) is usually a bad idea because it can lead to confusing test results. Instead, focus on mocking the external dependencies of your application.


3. Use `pytest.raises` for Exception Testing

Instead of mocking exceptions with conditional logic in your test, use `pytest.raises()` to check if a function raises the expected exception. This makes tests cleaner and more readable.

#### Example:

```python

def test_invalid_operation():
    with pytest.raises(ValueError, match="Invalid operation"):
        invalid_operation()
```

This ensures that the test passes only if `invalid_operation` raises a `ValueError`.

### Conclusion

Mocking is a powerful technique that simplifies testing by isolating the code under test from its dependencies. `Pytest` offers excellent support for mocking through its built-in `monkeypatch` fixture and integration with `unittest.mock`. Whether you’re mocking functions, classes, or even asynchronous code, pytest’s flexibility ensures your tests remain clean, fast, and reliable. By following best practices, you can maintain a robust test suite that is easy to manage and scale as your project grows.