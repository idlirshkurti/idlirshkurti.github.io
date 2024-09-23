---
layout: page  
title: FastAPI
date: 2024-09-22 15:05 +0100  
categories: [FastAPI, Tutorial]  
tags: [python, blog, fastapi pydantic]  
---

# A Simple Guide to FastAPI in Python
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

**Why Use FastAPI?**

FastAPI is a modern, high-performance Python web framework for building APIs quickly and efficiently. It's designed to be easy to use, with automatic data validation, documentation generation, and async support for handling thousands of requests per second.

**Key Features:**

1. **Fast**: Built on Python's async capabilities, it performs at the same level as Node.js and Go.
2. **Easy**: Clear, intuitive syntax.
3. **Data Validation**: Built-in request validation with Pydantic models.
4. **Automatic Documentation**: It generates interactive docs (Swagger, ReDoc) out of the box.

### Getting Started with FastAPI

#### Installation

To start using FastAPI, you first need to install it:


```bash
pip install fastapi
pip install "uvicorn[standard]"
```

Here, FastAPI is the main library, and Uvicorn is an ASGI server used to run the FastAPI app.

#### A Simple Example


```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello, FastAPI"}

@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

1. **`@app.get("/")`**: This decorator defines a GET endpoint at the root URL (`/`).
2. **`@app.get("/items/{item_id}")`**: Defines a dynamic URL path that accepts an integer parameter `item_id`.

#### Running the Application

To run this FastAPI application, use Uvicorn:


```bash
uvicorn main:app --reload
```

- **`main:app`**: Refers to the `app` instance in the `main.py` file.
- **`--reload`**: Enables auto-reloading on code changes.

FastAPI provides interactive API documentation out-of-the-box at `/docs` (Swagger UI) and `/redoc` (ReDoc).

### Using `Pydantic` Models for Data Validation

FastAPI integrates seamlessly with `Pydantic` for data validation. You can define request and response models that enforce data types and structure.

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool = None

@app.post("/items/")
def create_item(item: Item):
    return {"item_name": item.name, "item_price": item.price}
```

Here, the **`Item`** model ensures that incoming request data conforms to the specified structure.

### Asynchronous Support

FastAPI allows defining async functions to improve performance for I/O-bound operations, such as database queries.

```python
@app.get("/async/")
async def read_async():
    return {"message": "This is asynchronous"}
```

### Conclusion

FastAPI simplifies API development by providing features like automatic documentation, built-in validation, and async capabilities. Whether you're building a small microservice or a complex API, FastAPI is a great choice for Python developers who value performance and ease of use.