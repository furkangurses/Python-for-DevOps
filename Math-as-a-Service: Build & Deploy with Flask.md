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
