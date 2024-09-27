---
layout: page
title: Justfile
parent: Task Automation
description: Using justfiles in python projects
nav_order: 1
tags: [python, justfile, tasks]
---

# Justfile: Streamline Your Workflow with Simple, Powerful Automation
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

A `Justfile` is a modern, user-friendly alternative to `Makefile` for managing project tasks. While `Makefile` is widely used, `justfile` (managed via the `just` command-line tool) offers a cleaner syntax, better error messages, and more powerful features like argument passing and simple variable management. It is ideal for automating common project tasks such as running tests, setting up environments, and performing deployments.

In this guide, we’ll go through how to set up and use a `Justfile` in a Python project and compare it to traditional `Makefile` usage.
---

### 1. Why Use Justfile Over Makefile?

While `Makefile` is ubiquitous in task automation, it has several limitations:
- **Strict Syntax**: `Makefile` requires indentation by tabs and is prone to syntax errors.
- **Lack of Modern Features**: `Makefile` lacks features like argument passing, simple conditionals, or built-in shell improvements.
- **Cross-Platform Issues**: `Makefile` works best on Unix-like systems and often requires additional tools to function on Windows.

In contrast, `justfile` offers several advantages:
- **Better Syntax**: `Justfile` has a more forgiving and readable syntax, using spaces for indentation.
- **Cross-Platform**: Works smoothly on both Unix-like systems and Windows without additional setup.
- **Arguments and Variables**: It natively supports passing arguments to tasks, making it more flexible for complex workflows.
- **Error Reporting**: Provides better error messages and more readable output.

### 2. Setting Up Just

First, install `just` by following the installation instructions for your platform. On macOS or Linux, you can install `just` with Homebrew or download the binary directly:

```bash
brew install just
```

On Ubuntu or Debian:

```bash
sudo apt install just
```

For Windows, download the binary from the [Just GitHub Releases](https://github.com/casey/just/releases) page or use Scoop:

```bash
scoop install just
```

Once installed, you can verify the installation:

```bash
just --version
```

### 3. Creating a Justfile

A `Justfile` defines the tasks for your project. It is similar in function to a `Makefile` but more readable and flexible. Below is a basic example:

```justfile
# Justfile for a Python Project

# Define a virtual environment directory
venv_dir := "venv"

# Task: Create a virtual environment
venv:
    python3 -m venv {{venv_dir}}
    @echo "Virtual environment created in {{venv_dir}}."

# Task: Install dependencies
install:
    {{venv_dir}}/bin/pip install -r requirements.txt
    @echo "Dependencies installed."

# Task: Run tests
test:
    {{venv_dir}}/bin/pytest

# Task: Lint the codebase
lint:
    {{venv_dir}}/bin/flake8 .

# Task: Format the codebase
format:
    {{venv_dir}}/bin/black .

# Task: Clean the environment
clean:
    rm -rf {{venv_dir}}
    @echo "Cleaned up the virtual environment."
```

### 4. Common Python Project Tasks

You can use a `Justfile` to automate various tasks in your Python project, from dependency management to running tests.

#### 4.1. Installing Dependencies

Define a task for installing project dependencies with `pip` or `poetry`:

```justfile
install:
    pip install -r requirements.txt
    @echo "Dependencies installed."
```

#### 4.2. Running Tests

Run tests using `pytest` or another test framework:

```justfile
test:
    pytest
```

#### 4.3. Linting and Formatting

You can add tasks for linting and code formatting, as well:

```justfile
lint:
    flake8 .

format:
    black .
```

#### 4.4. Managing Virtual Environments

Create and activate a virtual environment using:

```justfile
venv:
    python3 -m venv venv
    @echo "Virtual environment created."
```

You can also combine multiple tasks in a single command. For example, to create a virtual environment and install dependencies in one go:

```justfile
setup: venv install
    @echo "Project setup complete."
```

### 5. Advanced Features of Just

`justfile` provides advanced features that enhance flexibility, such as argument passing, conditional logic, and default variables.

#### Passing Arguments to Tasks

You can pass arguments to a task:

```justfile
# Task with argument
greet name:
    @echo "Hello, {{name}}!"
```

Run it like this:

```bash
just greet Alice
```

#### Default Variables

Define variables at the top and reuse them across tasks:

```justfile
venv_dir := "venv"

# Using the venv_dir variable
install:
    {{venv_dir}}/bin/pip install -r requirements.txt
```

#### Conditional Execution

You can use simple conditional statements to check for the existence of files or directories:

```justfile
# Task to ensure virtual environment exists
venv:
    if [ ! -d "{{venv_dir}}" ]; then
        python3 -m venv {{venv_dir}};
    fi
```

### 6. Differences Between Justfile and Makefile

Here’s a breakdown of key differences between `Justfile` and `Makefile`:

| Feature                   | Justfile                                | Makefile                                           |
| ------------------------- | --------------------------------------- | -------------------------------------------------- |
| **Language**              | Shell-like, but simplified syntax       | Traditional Shell commands                         |
| **Cross-Platform**        | Cross-platform, works better on Windows | More tools needed for Windows                      |
| **Ease of Use**           | Easier to read, write, and maintain     | Can be complex with advanced shell usage           |
| **Error Reporting**       | More descriptive error messages         | Basic shell error reporting                        |
| **Task Parameters**       | Simple syntax for parameters            | Requires shell argument passing                    |
| **Task Dependencies**     | Supports dependencies and task chaining | Supports dependencies, used often in build systems |
| **File Compilation**      | Less focus on file compilation          | Commonly used for building files                   |
| **Shell Integration**     | Can run shell commands directly         | Direct integration with shell commands             |
| **Dependency Management** | No built-in, relies on external tools   | Not designed for dependency management             |

### Example: A Justfile vs. Makefile

Here’s an example comparing `Justfile` and `Makefile` for the same task.

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

In the `Justfile`:
- No need to use `$(VENV_DIR)`—`{{venv_dir}}` is easier to read.
- Recipes in `Justfile` use spaces instead of tabs.
- Variables are declared with `:=`, which is simpler and less error-prone.

### 7. Conclusion

`Justfile` offers a modern and intuitive approach to task automation in Python projects. It simplifies many of the complexities associated with `Makefile`, providing a cleaner, more user-friendly experience. By supporting features like argument passing, conditionals, and better error handling, `Justfile` is a great choice for both small and large Python projects.

**Key Benefits of Justfile**:
- **Easier syntax** with space-based indentation and no reliance on tabs.
- **Cross-platform support**, making it ideal for developers using different operating systems.
- **Better error messages** and readability.
- **Built-in features** like argument passing, which reduces the need for custom shell scripts.

For Python developers, `Justfile` is a more flexible and modern tool than `Makefile`, while still offering the same task automation power.