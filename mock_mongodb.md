---
layout: page
title: Database mocking
parent: Pytest
---

# Mocking MongoDB in `Pytest` with Asynchronous Querying

Testing database-related code can be tricky, especially when working with asynchronous operations like those in MongoDB. However, mocking a MongoDB database during tests can help you focus on your logic without depending on the actual database.

In this guide, we’ll cover how to mock MongoDB in pytest with `motor`, an asynchronous Python driver for MongoDB. We’ll explore mocking the database connection, queries, and testing the results in asynchronous contexts.

## Why Mock MongoDB?

When testing a system that interacts with MongoDB, setting up a real database can add complexity to your tests. Mocking the database allows you to:

- Isolate your test from external dependencies.
- Simulate different data scenarios.
- Run tests faster without needing a live database.

## Tools You Need

To follow along, make sure you have the following tools in your Python environment:

- **pytest**: The testing framework.
- **pytest-asyncio**: To handle async tests in pytest.
- **motor**: An asynchronous driver for MongoDB.
- **unittest.mock**: The standard mocking library.

Here’s how to set it up:

```bash
pip install pytest pytest-asyncio motor
```

### Basic Setup with Motor and Pytest

First, let’s create a basic setup where we define an async function to query MongoDB:


```python

from motor.motor_asyncio import AsyncIOMotorClient

async def get_user_data(user_id: str):
    client = AsyncIOMotorClient('mongodb://localhost:27017')
    db = client['mydatabase']
    collection = db['users']
    user_data = await collection.find_one({"_id": user_id})
    return user_data
```

This function connects to a MongoDB instance, fetches user data by `user_id`, and returns the result.

### Mocking MongoDB with `unittest.mock`

We want to test `get_user_data()` without interacting with a real MongoDB instance. For this, we’ll mock the Motor client, the collection, and the query.

Here’s how to do it with pytest.

#### Step 1: Mock the MongoDB Client

We’ll use `unittest.mock.patch` to mock the Motor client and its methods. We can patch the `AsyncIOMotorClient` so that it doesn’t connect to the real MongoDB.


```python
from unittest.mock import patch, AsyncMock

@patch('motor.motor_asyncio.AsyncIOMotorClient')
async def test_get_user_data(mock_client):
    # Mock the database and collection
    mock_db = AsyncMock()
    mock_collection = AsyncMock()
    
    # Set up the mock client
    mock_client.return_value.__getitem__.return_value = mock_db
    mock_db.__getitem__.return_value = mock_collection

    # Mock the find_one() query result
    mock_collection.find_one.return_value = {"_id": "123", "name": "John Doe"}

    # Call the function to test
    result = await get_user_data("123")
    
    # Assertions
    assert result == {"_id": "123", "name": "John Doe"}
    mock_collection.find_one.assert_called_once_with({"_id": "123"})

```

#### Step 2: Breaking Down the Mocking Process

- `@patch('motor.motor_asyncio.AsyncIOMotorClient')`: This decorator replaces the real `AsyncIOMotorClient` with a mock version.
- `mock_client.return_value.__getitem__.return_value`: This handles accessing the database and collection as you would with a real client. The `__getitem__` method is used when accessing databases and collections (e.g., `client['mydatabase']`).
- `mock_collection.find_one.return_value`: Mocks the result of the MongoDB query.
- `mock_collection.find_one.assert_called_once_with({"_id": "123"})`: Ensures the query is called with the correct parameters.

### Testing Asynchronous Queries

When testing asynchronous functions, it’s important to use `pytest-asyncio`. This allows pytest to handle `await` statements within the test function. Here’s a step-by-step breakdown of how it works:

#### Step 3: Using `pytest-asyncio`


```python
import pytest

@pytest.mark.asyncio
@patch('motor.motor_asyncio.AsyncIOMotorClient')
async def test_get_user_data(mock_client):
    mock_db = AsyncMock()
    mock_collection = AsyncMock()
    
    mock_client.return_value.__getitem__.return_value = mock_db
    mock_db.__getitem__.return_value = mock_collection

    mock_collection.find_one.return_value = {"_id": "123", "name": "John Doe"}

    result = await get_user_data("123")
    
    assert result == {"_id": "123", "name": "John Doe"}

```

The key to this test is `@pytest.mark.asyncio`, which tells pytest to run the function asynchronously. Without this decorator, the `await` statement would raise an error.

### Mocking Multiple Database Queries

Sometimes, your function may execute more than one query. In this case, you can mock each query separately:

```python
async def get_full_user_data(user_id: str):
    client = AsyncIOMotorClient('mongodb://localhost:27017')
    db = client['mydatabase']
    collection = db['users']
    user_data = await collection.find_one({"_id": user_id})
    user_profile = await collection.find_one({"user_id": user_id, "type": "profile"})
    return user_data, user_profile
```

To test this, mock both `find_one()` queries:

```python
@patch('motor.motor_asyncio.AsyncIOMotorClient')
async def test_get_full_user_data(mock_client):
    mock_db = AsyncMock()
    mock_collection = AsyncMock()

    mock_client.return_value.__getitem__.return_value = mock_db
    mock_db.__getitem__.return_value = mock_collection

    # Mock multiple queries
    mock_collection.find_one.side_effect = [
        {"_id": "123", "name": "John Doe"},
        {"user_id": "123", "type": "profile", "bio": "Software Engineer"}
    ]

    result = await get_full_user_data("123")
    
    assert result == (
        {"_id": "123", "name": "John Doe"},
        {"user_id": "123", "type": "profile", "bio": "Software Engineer"}
    )

```

Here, `mock_collection.find_one.side_effect` allows us to return different results for each query call.

### Mocking Exceptions

Sometimes, you need to test how your function behaves when an exception occurs during a database query. You can use `side_effect` to simulate exceptions.

```python
@patch('motor.motor_asyncio.AsyncIOMotorClient')
async def test_get_user_data_raises_exception(mock_client):
    mock_db = AsyncMock()
    mock_collection = AsyncMock()

    mock_client.return_value.__getitem__.return_value = mock_db
    mock_db.__getitem__.return_value = mock_collection

    # Simulate an exception on find_one
    mock_collection.find_one.side_effect = Exception("Database error")

    with pytest.raises(Exception, match="Database error"):
        await get_user_data("123")
```

This test verifies that your function handles exceptions correctly when a MongoDB query fails.

---

## Conclusion

Mocking MongoDB with pytest for asynchronous operations ensures your tests are fast, isolated, and reliable. By using tools like `unittest.mock` and `pytest-asyncio`, you can efficiently test your MongoDB-dependent code without needing a live database. This approach lets you focus on testing the core logic of your functions while keeping your tests clean and predictable.