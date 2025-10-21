

# Developer Guide -  Building \& Registering Brane Packages

_Last updated: October 2025_  
**Audience:** Software engineers creating custom Brane functions  
**Purpose:** Learn how to wrap your code into reusable Brane packages and register them in Brane.  

---

##  1. Overview

A **Brane package** is a portable containerized unit that wraps any code (e.g., Python, R, shell) and exposes it to the Brane ecosystem as a callable function.  
Packages allow **reusability**, **language independence**, and **safe execution** in local or distributed environments.

When you create a Brane package, you:
1. Write your function code  
2. Describe it using metadata (`container.yml`, `data.yml`)  
3. Build it into a container with the Brane CLI  
4. Register (push) it so others can use it in BraneScript workflows  

---

## 2. Package Basics

A Brane package contains:
 ```css
 my_package/
  |---- container.yml # Function definitions and metadata
  |---- data.yml # (Optional) Data types used by the package
  |---- code/ # Source code directory
  |  |---- main.py
  |---- requirements.txt # (Optional) dependencies
```

yaml
 `container.yml`

Defines **what functions** your package exposes and how to run them.

Example:
```yaml
name: hello_world
version: 1.0.0
language: python
functions:
  say_hello:
    command: python3 main.py
    description: "Prints a friendly message" 
```

data.yml (Optional)
Defines custom data types exchanged between packages.

Example:
```yaml
types:
  Message:
    fields:
      content: string
```

## 3. Creating Your First Brane Function
### Step 1  -  Create a New Package Skeleton
Use the brane CLI to generate a template:

```bash
brane new hello_world
cd hello_world
This creates the following structure:
```

```css
hello_world/
 |---- container.yml
 |---- data.yml
 |----  code/
```

### Step 2  - Add Your Function Code
Create a Python script in the code/ directory, for example main.py:

```python
# code/main.py
def say_hello():
    print("Hello, world!")
```

### Step 3 - Define the Interface
Edit the generated container.yml file:

```yaml
name: hello_world
version: 1.0.0
language: python
functions:
  say_hello:
    command: python3 code/main.py
    description: "Prints a simple Hello, world! message"
```
If your function requires arguments, you can specify them:

```yaml
functions:
  greet:
    command: python3 code/greet.py
    inputs:
      name: string
    output: string
```
Example Python code:

```python
# code/greet.py
import sys

def greet(name):
    return f"Hello, {name}!"

if __name__ == "__main__":
    name = sys.argv[1]
    print(greet(name))
```
## 4. Building the Package
Once your files are ready, build the package into a container:

```bash
brane build
```
Brane will:

Create a Docker image for your package

Validate your metadata

Store the resulting .tar artifact under .brane/

You can inspect the result:

```bash
brane inspect hello_world:1.0.0
```
##  5. Pushing (Registering) the Package
To make your package available in your local or shared Brane registry:

```bash
brane push
```
This uploads the built package to the Brane registry so it can be imported in workflows.

Check it is available:

```bash
brane packages list
```
You should see:

```pgsql

NAME          VERSION   OWNER
hello_world   1.0.0     local
```
## 6. Testing Locally
You can test your function without writing a BraneScript yet.

Run your package directly:

```bash

brane run hello_world:say_hello
```
Expected output:

```
Hello, world!
```
If the package takes input arguments:

```bash
brane run hello_world:greet --input name="Ada"
Output:
```

Hello, Ada!

## 7. Using the Package in a Workflow
Once registered, import and use it in a BraneScript workflow.

Create a file hello.bs:

```branescript
import hello_world;

println(hello_world::say_hello());
```
Run the workflow:

```bash
brane workflow run hello.bs
```

## 8. Tips  \& Best Practices
 Version your packages - increment the version every time you modify code or interfaces.
  Keep metadata simple -  avoid unnecessary dependencies.
  Use consistent naming - package names should match their primary function.
  Document functions - use description fields for discoverability.
  Test locally before pushing to shared registries.

## Summary
### Step	Command	Purpose
Create package | brane new <name> | Generate skeleton |
| ---- | -------------------- | ------------------------------------ |
Edit metadata | container.yml, data.yml | Define interfaces and functions | 
Build package | brane build | Create container image |  
Push package | brane push | Register package in registry | 
Run or test | brane run | Verify function output | 
Use in workflow	brane workflow run | Integrate in BraneScript | 

You have now created, built, and registered your first Brane package!

Next steps:

Learn BraneScript  

Run HelloWorld workflows   

 [? Back to Table of Contents](Brane-documentation-index.md)

---

TODO:  generate the **BraneScript User Guide** next (to complete the user–developer–workflow triad)? It would explain how to write and run scripts that use these packages.