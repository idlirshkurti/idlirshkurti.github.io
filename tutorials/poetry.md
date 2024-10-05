---
layout: page  
title: Poetry
parent: the-dev-hub
date: 2024-09-22 15:05 +0100  
categories: [Poetry, Tutorial]  
tags: [python, blog, poetry, repository, dependencies]  
---

# Manage your python project with Poetry
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Poetry is a powerful tool that simplifies the management of Python projects, offering a unified interface for dependency management, packaging, and project setup. Whether you are managing simple scripts or complex libraries, Poetry can drastically improve your workflow by handling virtual environments, versioning, and package distribution seamlessly.

---

### 1. Why Do We Need Poetry?

Dependency management and environment configuration can be complex in Python projects, particularly when working with multiple libraries or distributing projects. Poetry solves several common problems:

- **Dependency Management**: Traditional methods of dependency management (like `requirements.txt` or manually editing `setup.py`) can become cumbersome when managing large projects. Poetry automates dependency resolution and ensures that your project’s dependencies are isolated and reproducible.
- **Virtual Environments**: Poetry automates the creation and activation of virtual environments, eliminating the need for tools like `virtualenv` or `pyenv`.
- **Project Packaging**: When distributing Python projects or libraries, Poetry simplifies packaging by providing default configurations and allowing you to publish directly to PyPI or other registries.
- **Reproducibility**: Poetry generates `poetry.lock` files that guarantee that the same versions of dependencies are installed across different environments.

### 2. Setting Up Poetry

Before using Poetry, you need to install it. Poetry supports multiple installation methods, but the most common and recommended method is via the official installer.

#### Installation:

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

You can verify that Poetry is installed by checking the version:

```bash
poetry --version
```

### 3. Creating and Managing a Project

Poetry makes it simple to create and configure Python projects. To create a new project, navigate to the desired directory and run the following command:

```bash
poetry new my_project
```

This command generates a basic project structure with the following files:
```
my_project/
├── pyproject.toml
├── README.rst
├── my_project/
│   └── __init__.py
└── tests/
    └── __init__.py
```

- **`pyproject.toml`**: The core configuration file for your project, which defines dependencies, project metadata, and settings.
- **`README.rst`**: A placeholder for your project’s documentation.
- **`my_project/`**: The main package where your code resides.
- **`tests/`**: A folder for organizing your project’s unit tests.

#### Converting Existing Projects

If you already have an existing project, you can initialize Poetry in that directory:

```bash
poetry init
```

This command will guide you through the process of creating the `pyproject.toml` file, where you can specify project metadata and dependencies.

### 4. Managing Dependencies

Poetry automatically handles dependency resolution and version compatibility. It creates a `pyproject.toml` file that tracks your project’s requirements and a `poetry.lock` file that ensures consistent environments.

#### 4.1. Adding Dependencies

To add dependencies, you can use the `add` command:

```bash
poetry add pandas
```

This command adds `pandas` to your project’s `pyproject.toml` file and locks the version in the `poetry.lock` file. If you want to specify a version, you can do so:

```bash
poetry add pandas@1.3.3
```

#### 4.2. Removing Dependencies

If you no longer need a package, remove it with the `remove` command:

```bash
poetry remove pandas
```

Poetry will update both the `pyproject.toml` and `poetry.lock` files accordingly.

#### 4.3. Using Dev Dependencies

Dev dependencies are packages that you only need for development (e.g., testing frameworks, linters). To add a development dependency:

```bash
poetry add pytest --group dev
```

You can also add dependencies for other groups, such as `test`, `doc`, or custom categories to organize different types of dependencies.

### 5. Virtual Environment Management

One of Poetry’s key features is its automatic handling of virtual environments. You don’t have to manually create and activate virtual environments.

- To install dependencies in a virtual environment, simply run:

    ```bash
    poetry install
    ```

- To activate the virtual environment and work inside it:

    ```bash
    poetry shell
    ```

- To deactivate, just type `exit`.

If you ever need to view where the virtual environment is located, use:

```bash
poetry env info
```

You can customize virtual environment behavior by modifying `poetry.toml`, but the default settings usually suffice.

### 6. Project Versioning and Packaging

Poetry makes it easy to version your project by defining the version in the `pyproject.toml` file.

```toml
[tool.poetry]
name = "my_project"
version = "0.1.0"
description = "A sample Python project."
```

- **Semantic Versioning**: Poetry encourages semantic versioning (`major.minor.patch`), which helps in maintaining compatibility between different versions of your project.

You can change the version using the command:

```bash
poetry version patch  # Updates the patch number
poetry version minor  # Updates the minor version
poetry version major  # Updates the major version
```

### 7. Building and Publishing Python Packages

Once your project is ready for distribution, Poetry simplifies the build and publish process.

#### Building Your Project

To package your project into distributable formats (such as `.tar.gz` and `.whl`):

```bash
poetry build
```

This creates the distribution files in the `dist/` directory.

#### Publishing Your Package

To publish your package to PyPI (or a private registry), first, ensure you’ve set up your account credentials:

```bash
poetry publish --build
```

You can also publish to a custom registry by adding the `--repository` flag.

### 8. Conclusion

Poetry significantly simplifies the management of Python projects by unifying dependency management, virtual environment creation, packaging, and publishing. By streamlining common tasks, it ensures that your project remains organized, reproducible, and easy to share or distribute.

**Why Use Poetry:**
- Automated dependency resolution and conflict detection.
- Integrated virtual environment management.
- Simplified project packaging and versioning.
- Ensures reproducibility with `pyproject.toml` and `poetry.lock` files.

Incorporating Poetry into your Python workflow will save time and prevent many common pitfalls related to dependencies, environments, and project distribution.
