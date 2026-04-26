# Math-as-a-Service: Build & Deploy with Flask

This project demonstrates the end-to-end lifecycle of a web application: from creating a standalone mathematical Python package to deploying it via a Flask web interface.

----------

## 📑 Project Architecture

The application is structured using a modular approach. The logic is separated into a dedicated package (`Maths`), keeping the server entry point clean and maintainable.

----------

## 🛠 Task 1: Building the Logic (The `Maths` Package)

First, we encapsulate the core mathematical logic inside a directory. This allows the functions to be reused across different projects.

### `Maths/mathematics.py`

Python

```
def summation(a, b):
    return a + b

def subtraction(a, b):
    return a - b

def multiplication(a, b):
    return a * b

```

### `Maths/__init__.py`

This file treats the directory as a package, allowing for clean imports.

Python

```
from . import mathematics

```

----------

## 🚀 Task 2 & 3: Web Deployment (`server.py`)

In this phase, we map HTTP endpoints to our package functions. We use `request.args` to capture user input and `render_template` to serve the UI.

### The Integration Logic

One key requirement is the **Integer Validation**. Since math operations with floats often return `.0` (e.g., `5.0`), we use `.is_integer()` to clean the output for the user.

Python

```
from flask import Flask, render_template, request
from Maths.mathematics import summation, subtraction, multiplication

app = Flask(__name__)

@app.route("/")
def render_index_page():
    return render_template('index.html')

@app.route("/sum")
def sum_route():
    num1 = float(request.args.get('num1'))
    num2 = float(request.args.get('num2'))
    result = summation(num1, num2)
    
    # Return integer if whole number, else return float string
    if result.is_integer():
        return str(int(result))
    return str(result)

@app.route("/sub")
def sub_route():
    num1 = float(request.args.get('num1'))
    num2 = float(request.args.get('num2'))
    result = subtraction(num1, num2)
    return str(int(result)) if result.is_integer() else str(result)

@app.route("/mul")
def mul_route():
    num1 = float(request.args.get('num1'))
    num2 = float(request.args.get('num2'))
    result = multiplication(num1, num2)
    return str(int(result)) if result.is_integer() else str(result)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)

```

----------

## 🧪 Deployment & Testing

To launch the application in the laboratory environment:

1.  **Initialize the Server:**
    
    Bash
    
    ```
    python3.11 server.py
    
    ```
    
2.  **Launch UI:** Connect to port `8080` via the Skills Network Toolbox.
    
3.  **Manual API Test:**
    
    Bash
    
    ```
    curl "localhost:8080/sum?num1=10&num2=5"
    # Output: 15
    
    ```
    

----------

## 💡 Practice Exercise: Robust Error Handling

Furkan, labın sonundaki "Optional" kısmı senin için çok önemli. Kullanıcı sayı yerine harf girerse sunucun `500 Internal Server Error` verir ve çöker. Bunu engellemek için şu yapıyı ekleyebilirsin:

### Defensive Programming Pattern

Python

```
@app.route("/sum")
def safe_sum():
    try:
        num1 = float(request.args.get('num1'))
        num2 = float(request.args.get('num2'))
        return str(summation(num1, num2))
    except (TypeError, ValueError):
        return {"message": "Invalid input. Please provide numbers."}, 400
```


---



## 🛠 The Developer’s Toolkit: Key Flask Utilities

To build a functional CRUD app, you need more than just routes. You need to control how the user moves through the application.

### 1. The Data Bridge: `request.form`

When a user types into an HTML input and hits "Submit," that data is packaged into a POST request. Flask captures this in a dictionary-like object called `request.form`.

-   **HTML:** `<input name="username">`
    
-   **Python:** `username = request.form['username']`
    

### 2. The Dynamic GPS: `url_for` and `redirect`

Hardcoding URLs (like `redirect('/login')`) is a recipe for broken links if you ever change your route names.

-   **`url_for('function_name')`**: Finds the URL associated with a specific function.
    
-   **`redirect()`**: Sends the user to that new location.
    

> **Pro Tip:** Always use them together: `return redirect(url_for('home'))`.

----------

## 🔄 The CRUD Lifecycle

### 🟢 Create (The "POST" logic)

Creating a record usually requires two steps in one route:

1.  **GET:** Display the empty form to the user.
    
2.  **POST:** Grab the data from `request.form`, save it, and redirect the user.
    

### 🔵 Read (The "Dynamic" logic)

To read a specific item, we use variable rules in the route, such as `@app.route('/view/<int:id>')`. This allows a single function to handle thousands of unique items based on their ID.

### 🟡 Update (The "Hybrid" logic)

Updating is the most complex step because it combines **Read** and **Create**:

1.  **GET:** Fetch the existing record from the database and "pre-fill" the HTML form so the user sees what they are editing.
    
2.  **POST:** Capture the new changes and overwrite the old record.
    

### 🔴 Delete (The "Destructive" logic)

For security, deletion should almost always be a **POST** request (usually via a button in a small form). If you use a simple GET link for deletion, a search engine crawler or an accidental click could wipe out your data.

----------

## 📊 Summary of CRUD Mapping

**Operation**

**HTTP Method**

**Flask Route Component**

**Action**

**Create**

`POST`

`request.form`

Insert new data

**Read**

`GET`

`<int:id>`

Display data

**Update**

`POST`

`request.form` + `id`

Modify existing data

**Delete**

`POST`

`id`

Remove data
