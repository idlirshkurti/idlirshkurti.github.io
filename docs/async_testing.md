# Mocking Asynchronous Functions in Python

When testing Python’s `asyncio`-based functions, standard testing methods fall short due to async functions returning coroutine objects. This guide covers techniques to mock such functions, enabling efficient testing.

### Key Concepts
- **Futures**: Return async results, ideal for simulating async tasks.
- **`AsyncMock` (Python 3.8+)**: Simplifies mocking by eliminating the need to create `Future` objects manually.

### Examples
Basic mock:
```python
async def sum(x, y): 
    await asyncio.sleep(1)
    return x + y
```

Testing with pytest:

```python
import pytest
@pytest.mark.asyncio
async def test_sum(mock_sum):
    result = await sum(1, 2)
    assert result == 4
```
