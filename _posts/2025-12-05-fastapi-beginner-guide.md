---
layout: default
title: "FastAPI — Beginner Guide"
description: "A simple and practical introduction to FastAPI for beginners with examples, exercises, and project ideas."
date: 2025-12-05 06:01:42 -0500
tags: [python, fastapi, api, web, beginner]
categories: [howto, python, FastAPI, Coding]
excerpt: "A concise, practical FastAPI starter: install, first app, core concepts (path/query/body), CRUD example, exercises, and project ideas."
---

# FastAPI — Beginner Guide

A simple and practical introduction for beginners to build APIs with FastAPI.

Table of contents
- [What is FastAPI?](#what-is-fastapi)
- [Requirements](#requirements)
- [Installation](#installation)
- [Your First FastAPI App](#your-first-fastapi-app)
- [FastAPI Basics](#fastapi-basics)
  - [Path Parameters](#path-parameters)
  - [Query Parameters](#query-parameters)
  - [POST (JSON Body) with Pydantic](#post-json-body-with-pydantic)
- [CRUD Example (Practice)](#crud-example-practice)
- [Practice Exercises](#practice-exercises)
- [Project Ideas](#project-ideas)
- [FastAPI vs Litestar (Quick Comparison)](#fastapi-vs-litestar-quick-comparison)
- [Useful Links](#useful-links)
- [Conclusion](#conclusion)

---

## What is FastAPI? {#what-is-fastapi}

FastAPI is a modern, high-performance Python web framework for building APIs. It’s beginner-friendly, type-driven (uses Python type hints), and automatically generates interactive API docs (Swagger UI and ReDoc).

Great for:
- Building REST APIs
- Learning backend development
- Rapidly creating production-ready services

---

## Requirements {#requirements}

Before you start, you should know:
- Basic Python (functions, lists, dicts)
- How to install packages with pip
- Optional but recommended: virtual environments (venv, pipx, or poetry)

---

## Installation {#installation}

Create and activate a virtual environment (recommended):
```bash
python -m venv .venv
source .venv/bin/activate   # macOS / Linux
# .venv\Scripts\activate    # Windows (PowerShell)
```

Install FastAPI and Uvicorn (ASGI server):
```bash
pip install fastapi uvicorn
```

---

## Your First FastAPI App {#your-first-fastapi-app}

Create a file named `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Hello, FastAPI!"}
```

Run the app with auto-reload during development:

```bash
uvicorn main:app --reload
```

Open a browser:
- Swagger UI: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc

---

## FastAPI Basics {#fastapi-basics}

FastAPI maps Python functions to HTTP endpoints. It uses type hints to validate and document inputs and outputs.

### Path Parameters {#path-parameters}

Example: capture a user ID from the URL.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"id": user_id}
```

- `user_id` will be converted to `int` and validated automatically.

### Query Parameters {#query-parameters}

Example: search with query parameters and defaults.

```python
@app.get("/search")
def search(q: str = "", limit: int = 10):
    return {"query": q, "limit": limit}
```

- Access via: `/search?q=fastapi&limit=5`

### POST (JSON Body) with Pydantic {#post-json-body-with-pydantic}

Use Pydantic models to validate request bodies.

```python
from pydantic import BaseModel
from fastapi import FastAPI

app = FastAPI()

class User(BaseModel):
    name: str
    age: int

@app.post("/users")
def create_user(user: User):
    return user
```

- Request body must be JSON and match the `User` model.

---

## CRUD Example (Practice) {#crud-example-practice}

A simple in-memory example to practice CRUD endpoints. (Not for production.)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Dict

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None

items: Dict[int, Item] = {}
next_id = 1

@app.get("/items")
def list_items():
    return items

@app.post("/items")
def create_item(item: Item):
    global next_id
    items[next_id] = item
    response = {"id": next_id, **item.dict()}
    next_id += 1
    return response

@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"id": item_id, **items[item_id].dict()}

@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    items[item_id] = item
    return {"id": item_id, **item.dict()}

@app.delete("/items/{item_id}")
def delete_item(item_id: int):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    del items[item_id]
    return {"status": "deleted"}
```

---

## Practice Exercises {#practice-exercises}

Exercise 1 — Build CRUD for Users
- Implement POST/GET/PUT/DELETE for `/users` similar to the Items example. Use Pydantic models.

Exercise 2 — Filtering
- Add query parameters to filter results, e.g. `/users?name=John&age=30`.

Exercise 3 — Pagination
- Implement `/users?page=1&limit=5` and return paginated results.

Exercise 4 — Simple Token Auth
- Protect an endpoint by checking the `Authorization` header:
  - Header: `Authorization: Bearer SECRET123`
  - Use a dependency or middleware to validate tokens.

---

## Project Ideas {#project-ideas}

Beginner
- Todo API
- Notes API
- Employee API

Intermediate
- Blog API with comments and tags
- JWT Authentication (login/register)
- File upload service (images or documents)

Advanced
- Deploy FastAPI with Docker + Docker Compose or Kubernetes
- Add async DB access (SQLModel / async SQLAlchemy)
- Rate limiting, caching, and observability (logging + metrics)

---

## FastAPI vs Litestar (Quick Comparison) {#fastapi-vs-litestar-quick-comparison}

| Feature | FastAPI | Litestar |
|---|---:|:---|
| Learning Curve | ⭐ Easy | ⭐⭐ Medium |
| Documentation | ✔ Auto-generated | ✔ Auto-generated |
| Best For | Small–medium apps | Medium–large apps / advanced patterns |
| Popularity | Very high | Growing |

(Choose based on ecosystem, built-in features, and team familiarity.)

---

## Useful Links {#useful-links}

- FastAPI Official Docs: https://fastapi.tiangolo.com/
- FastAPI GitHub: https://github.com/tiangolo/fastapi
- Uvicorn: https://www.uvicorn.org/
- Pydantic: https://pydantic-docs.helpmanual.io/

---

## Conclusion {#conclusion}

FastAPI is an excellent entry point for API development in Python: fast to learn, productive, and with first-class auto-generated docs. Start with small CRUD projects, learn Pydantic models and dependencies, then scale to async DBs and deployments.

Happy coding! 🚀
