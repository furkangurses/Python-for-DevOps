# RESTful API Engineering with Flask

[![Lab Focus](https://img.shields.io/badge/Focus-API%20Architecture-purple.svg)](#)
[![Complexity](https://img.shields.io/badge/Complexity-Intermediate-yellow.svg)](#)
[![Testing](https://img.shields.io/badge/Testing-CURL-black.svg)](#)

This module transitions from basic routing to full RESTful API implementation. It covers explicit HTTP status management, dynamic URL parameters (`UUID`), JSON payload parsing (POST requests), and global error handlers.

---

## 1. Explicit Status Code Engineering (`make_response`)

While Flask defaults to returning `200 OK`, robust APIs require specific HTTP status codes to communicate exactly what happened.

### Implementation: Returning `204 No Content`
Use `204` when an operation succeeds but the server has no data to return (e.g., successful deletion without returning the deleted object).
```python
@app.route("/no_content")
def no_content():
    """Returns a tuple indicating successful execution without a body payload."""
    return ({"message": "No content found"}, 204)

```

### Implementation: The `make_response` Utility

For maximum control over the headers and status, construct the response object manually.

Python

```
from flask import make_response

@app.route("/exp")
def index_explicit():
    resp = make_response({"message": "Hello World"})
    resp.status_code = 200
    return resp

```

----------

## 2. Query Parameter Processing (`request.args`)

APIs often filter data based on URL query strings (e.g., `?q=search_term`).

### Implementation: Search Endpoint

This endpoint extracts the `q` argument to filter an in-memory dictionary.

Python

```
from flask import request

@app.route("/name_search")
def name_search():
    # 1. Extract query parameter
    query = request.args.get("q")

    # 2. Validation (Check for null or empty/numeric inputs)
    if query is None:
        return {"message": "Query parameter 'q' is missing"}, 400
    if query.strip() == "" or query.isdigit():
        return {"message": "Invalid input parameter"}, 422

    # 3. Execution (Search logic)
    for person in data:
        if query.lower() in person["first_name"].lower():
            return person, 200

    # 4. Fallback (Not found)
    return {"message": "Person not found"}, 404

```

----------

## 3. RESTful Resource Routing (Dynamic URLs)

A proper REST architecture embeds identifiers directly into the URL path. Here we enforce the identifier to be a Valid `UUID`.

### Handling GET Requests (Read Resource)

Python

```
@app.route("/person/<uuid:id>")
def find_by_uuid(id):
    for person in data:
        if person["id"] == str(id):
            return person, 200
    return {"message": "person not found"}, 404

```

### Handling DELETE Requests (Remove Resource)

Notice the explicit declaration of `methods=['DELETE']`.

Python

```
@app.route("/person/<uuid:id>", methods=['DELETE'])
def delete_by_uuid(id):
    for person in data:
        if person["id"] == str(id):
            data.remove(person)
            return {"message": f"Person with ID {id} deleted"}, 200
    return {"message": "person not found"}, 404

```

----------

## 4. Payload Parsing (Handling POST Requests)

When clients send data to create a new resource, it arrives in the request body as JSON.

### Implementation: Creating a Resource

Python

```
@app.route("/person", methods=['POST'])
def add_by_uuid():
    # 1. Parse JSON payload
    new_person = request.json
    
    # 2. Basic Validation
    if not new_person:
        return {"message": "Invalid input parameter"}, 422
        
    try:
        # 3. State Mutation (Adding to mock database)
        data.append(new_person)
    except NameError:
        return {"message": "Data store unavailable"}, 500

    # 4. Confirmation
    return {"message": f"{new_person['id']}"}, 200

```

**Test via CURL:**

Bash

```
curl -X POST -H 'Content-Type: application/json' -d '{"id": "123", "first_name": "Furkan"}' http://localhost:5000/person

```

----------

## 5. Global Error Handlers (Application Resilience)

By default, Flask returns HTML pages for `404` and `500` errors. In a microservices architecture, **every response must be predictable JSON.**

### Overriding Default Error Pages

Use `@app.errorhandler` to catch exceptions globally and format them appropriately.

Python

```
@app.errorhandler(404)
def api_not_found(error):
    """Overrides default HTML 404 with a JSON structure."""
    return {"message": "API not found"}, 404

@app.errorhandler(500)
def server_error(error):
     """Ensures internal crashes don't leak stack traces to the client."""
     return {"message": "Internal Server Error"}, 500
```
