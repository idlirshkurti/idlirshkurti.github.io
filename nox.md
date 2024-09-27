---
layout: page
title: Nox
parent: Task Automation
description: Using nox in python projects
nav_order: 1
tags: [python, nox, tasks]
---

# Nox: Simplify Your Python Workflow with Task Automation
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Nox is a powerful automation tool designed specifically for Python projects. Similar to `Makefile` and `Justfile`, it allows you to automate tasks like testing, linting, and dependency management. However, Nox offers a Pythonic approach to task automation with a focus on testing in multiple Python environments, which is a feature that makes it particularly well-suited for continuous integration (CI) pipelines.

In this guide, we’ll cover how to set up and use Nox in your Python projects and explain why it’s a great choice for developers, especially in testing scenarios.

---

### 1. Why Use Nox Over Makefile or Justfile?

While `Makefile` and `Justfile` are great for general task automation, Nox focuses on simplifying the automation of Python-specific tasks, especially testing and virtual environment management.

Here are some reasons why you might choose Nox over other tools:
- **Pythonic**: Nox allows you to write tasks in Python, making it easier for Python developers to manage.
- **Built-in Virtual Environment Management**: Nox automatically manages virtual environments, so you don’t have to handle them manually.
- **Multi-Python Version Testing**: Nox makes it easy to test your code in multiple Python versions (e.g., 3.7, 3.8, 3.9) with minimal configuration.
- **Custom Sessions**: You can define different sessions for tasks like testing, linting, building, etc., in a modular way.

### 2. Setting Up Nox

To get started with Nox, you first need to install it. Nox can be installed via `pip`:

```bash
pip install nox
```

After installing Nox, you can check if it’s installed properly by running:

```bash
nox --version
```

Next, you’ll need to create a `noxfile.py` where you will define the sessions (tasks) for Nox to run.

### 3. Creating a Noxfile

A `noxfile.py` is where you define the tasks or "sessions" that Nox will execute. Here’s an example of a simple `noxfile.py`:

```python
import nox

# Session: Create a virtual environment and install dependencies
@nox.session
def install(session):
    session.install('pytest', 'flake8')

# Session: Run tests using pytest
@nox.session
def test(session):
    session.run('pytest')

# Session: Lint the code using flake8
@nox.session
def lint(session):
    session.run('flake8', '.')

# Session: Format code using black
@nox.session
def format(session):
    session.run('black', '.')
```

### 4. Common Python Project Tasks with Nox

You can use Nox to automate many tasks, including testing, linting, and virtual environment management.

#### 4.1. Running Tests

Nox makes it easy to run tests using frameworks like `pytest`. You simply define a session that runs the tests, and Nox will handle the virtual environment.

```python
@nox.session
def test(session):
    session.run('pytest')
```

To run the tests, execute:

```bash
nox -s test
```

#### 4.2. Managing Virtual Environments

Nox automatically handles the creation of virtual environments. You can define tasks that install dependencies within a virtual environment and run them in isolation from your system Python.

```python
@nox.session
def install(session):
    session.install('pytest', 'flake8')
```

Nox automatically ensures that each session runs in its own isolated environment.

#### 4.3. Linting and Formatting

You can define sessions for running linting tools like `flake8` or formatting tools like `black`:

```python
@nox.session
def lint(session):
    session.run('flake8', '.')

@nox.session
def format(session):
    session.run('black', '.')
```

Run the linting or formatting sessions with:

```bash
nox -s lint
nox -s format
```

#### 4.4. Testing in Multiple Python Versions

One of the standout features of Nox is its ability to test your code across multiple Python versions effortlessly. This is critical for ensuring compatibility across different environments.

```python
@nox.session(python=['3.7', '3.8', '3.9'])
def test(session):
    session.run('pytest')
```

With this configuration, Nox will create virtual environments for each Python version (if available on your system) and run the tests in each.

Run the tests in all Python versions with:

```bash
nox -s test
```

### 5. Advanced Features of Nox

Nox offers several advanced features that make it a powerful tool for Python projects:

#### Parametrized Sessions

You can pass parameters to sessions to customize their behavior:

```python
@nox.session
@nox.parametrize('pytest_args', ['--maxfail=1', '--maxfail=3'])
def test(session, pytest_args):
    session.run('pytest', pytest_args)
```

This session will run the tests with both `--maxfail=1` and `--maxfail=3` arguments.

#### Reusing Virtual Environments

By default, Nox recreates the virtual environment for each session. However, you can use the `--reuse-existing-virtualenvs` flag to speed up the process by reusing environments:

```bash
nox --reuse-existing-virtualenvs
```

#### Custom Virtual Environment Locations

If you want to specify a custom directory for your virtual environments, you can define it like this:

```python
@nox.session(venv_backend="virtualenv", venv_params=["--system-site-packages"])
def test(session):
    session.run('pytest')
```

### 6. Differences Between `Nox`, `Makefile` and `Justfile`

Here’s a breakdown of the differences between `Nox`, `Makefile` and `Justfile`:

| Feature                   | Nox                                      | Makefile                                     | Justfile                                                |
| ------------------------- | ---------------------------------------- | -------------------------------------------- | ------------------------------------------------------- |
| **Language**              | Python                                   | Shell (Makefile syntax)                      | Shell (simplified)                                      |
| **Virtual Environments**  | Automatic creation and management        | Requires manual handling                     | Requires manual handling                                |
| **Multi-Python Testing**  | Built-in                                 | Requires manual setup                        | Requires manual setup                                   |
| **Cross-Platform**        | Fully cross-platform                     | Needs additional tools for Windows           | Fully cross-platform, easier on Windows                 |
| **Error Reporting**       | Python exceptions, better feedback       | Basic shell error reporting                  | Similar to Makefile, but with clearer error messages    |
| **Ease of Use**           | Python-native, no shell scripting needed | Requires knowledge of shell scripting        | Simpler than Makefile, no special shell syntax required |
| **Task Parameters**       | Fully parameterized sessions with Python | Basic parameter passing via shell args       | Supports parameters with user-friendly syntax           |
| **File Management**       | Not required, focuses on automation      | Good for file compilation (e.g., `.o` files) | Less focused on file compilation                        |
| **Dependency Management** | Can manage Python dependencies via pip   | Not designed for dependency management       | No built-in package management, uses external tools     |

### Example: Nox vs. Makefile/Justfile

Here’s a side-by-side comparison of how you would define tasks for testing, linting, and dependency installation using Nox, `Makefile`, and `Justfile`.

**Makefile:**

```makefile
VENV_DIR = venv

venv:
	python3 -m venv $(VENV_DIR)

install:
	$(VENV_DIR)/bin/pip install -r requirements.txt
```

**Justfile:**

```justfile
venv_dir := "venv"

venv:
    python3 -m venv {{venv_dir}}

install:
    {{venv_dir}}/bin/pip install -r requirements.txt
```

**Noxfile:**

```python
import nox

@nox.session
def install(session):
    session.install('pytest', 'flake8')
```

The `noxfile.py` is written in Python, which is more intuitive for Python developers, while `Makefile` and `Justfile` rely on shell scripting.

### 7. Conclusion

Nox is a highly flexible, Python-native tool for automating tasks in Python projects. It excels in scenarios where you need to manage multiple Python versions, test environments, or complex workflows. Compared to `Makefile` and `Justfile`, Nox provides a more Pythonic and feature-rich approach, especially when it comes to managing virtual environments and running tests across different Python interpreters.

**Key Benefits of Using Nox**:
- **Pythonic Syntax**: Tasks are defined using Python, making it more accessible for Python developers.
- **Virtual Environment Management**: Built-in handling of virtual environments for isolated task execution.
- **Multi-Python Testing**: Easily test code across multiple Python versions.
- **Modular and Flexible**: Supports custom sessions and parameterization.

For Python-specific workflows, especially when testing across different environments and managing dependencies, Nox is a more powerful and flexible option than `Makefile` or `Justfile`.