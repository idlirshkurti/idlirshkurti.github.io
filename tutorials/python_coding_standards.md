---
layout: page
title: Coding Standards - Python
parent: Best Coding Practices
description: Best practices for writing python code
nav_order: 1
tags: [standards, devops, documentation]
---

# Writing High-Quality Python Code: Essential Standards and Guidelines
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

When working on Python projects, following a set of standardized guidelines can greatly improve code quality, readability, and maintainability. Whether you're scripting, building a library, or working on an application, these best practices will help you create cleaner, more efficient Python code.

---

## Python Version Compatibility

All code should be written in Python 3, with a minimum compatibility set to Python 3.7:

- **Avoid backward compatibility for Python 2**: There’s no need to make code compatible with Python 2 since it's no longer supported.
- **Library Development**: Libraries should ensure compatibility with Python 3.7 or later, allowing downstream applications to rely on stable, lower versions when necessary.
- **Application Development**: Applications are free to use any Python 3 version, as long as the dependencies align with it. For instance, if a feature requires Python 3.8, feel free to leverage it.
- **Testing for Libraries**: Libraries should list supported Python versions and be tested in CI against each of these versions.
- **Testing for Applications**: Applications should specify the required version and, optionally, test on newer versions as they become available.

---

## Build System

For project setup, follow the [PEP 518](https://www.python.org/dev/peps/pep-0518/) guidelines:

- Use `pyproject.toml` to define your build system requirements.
- With `setuptools`, leverage `setup.cfg` for metadata and dependency management.
- Optionally, include a basic `setup.py` file to support editable installs:
  ```python
  import setuptools

  if __name__ == "__main__":
      setuptools.setup()
  ```

---

## Structuring Your Project

The general structure for any Python project, whether a library or application, should be simple yet effective:

- **Source Code Layout**: Organize source code under a `src/` directory. For a project with multiple modules, this can look like:
  ```
  src/
    main_package/
      __init__.py
      module_one.py
  ```
- **Testing Layout**: Place tests within a `tests/` directory, with subfolders mirroring the structure of the code they test.
  ```
  tests/
    test_main_package/
      test_module_one.py
  ```

---

## Code Style Guidelines

Adhere to the [PEP 8 Style Guide](https://www.python.org/dev/peps/pep-0008/) for consistency:

- **Formatting and Linting**: Use tools like `black` for deterministic formatting and ensure compliance in CI.
- **Imports**: Use the [Google import style](https://google.github.io/styleguide/pyguide.html#22-imports), which is compatible with PEP 8 and structures imports as:
  1. Standard library imports
  2. Third-party imports
  3. Local imports (separate groups with a blank line)
- **Variable Naming**: Opt for descriptive names that make the code self-explanatory.
- **Avoid Commented-Out Code**: Only include necessary code in patches or pull requests—avoid leaving in commented-out sections.

---

## Type Annotations

For better readability and maintainability, all functions should include type hints for parameters and return values:

- **Type-Checking**: Use [mypy](http://www.mypy-lang.org/) in CI to validate type hints.
- **Void Functions**: Include a return type of `-> None` if the function has no return value.
- **Library Type Information**: Follow PEP 561 by including type information within packages and adding a `py.typed` marker file.

---

## Documentation Standards

Documentation helps others understand and use your code effectively:

- **Function Docstrings**: Include detailed explanations of parameters and return values.
- **Class Docstrings**: Explain class purpose and usage, describing each field when appropriate.
- **Module-Level Documentation**: At the top of each module, provide an overview of the contents and their intended usage.
- **Avoid Redundancies**: Don’t duplicate information (such as type hints) in docstrings, as this can become outdated.

---

## Testing Guidelines

Quality code is well-tested. For Python projects, follow these testing practices:

- **Testing Framework**: Use [pytest](https://docs.pytest.org).
- **Unit and Integration Tests**: Unit tests should cover individual functions, while integration tests validate the interactions between modules.
- **Async Testing**: If your code includes async functions, use the [`pytest-asyncio`](https://github.com/pytest-dev/pytest-asyncio) plugin.
- **Coverage**: Track code coverage with [`pytest-cov`](https://pytest-cov.readthedocs.io/en/latest/readme.html) and aim to cover branches as well.

---

## Setting Up a Testing Pipeline

Automating testing is crucial for consistency. Use [tox](https://tox.wiki/en/latest/) to define and manage your testing environment:

- **Isolated Build Environments**: Enable isolated builds by adding `isolated_build = true` to `tox.ini`.
- **Testing Stages**:
  - **Code Formatting**: `black`
  - **Documentation**: `pydocstyle` and `darglint`
  - **Code Standards**: `pylint`
  - **Type-Checking**: `mypy`
  - **Dependency Testing**: Run tests with minimum and latest dependency versions.

---

## Defining and Running Applications

For applications, utilize entry points for easy execution:

- **Entry Point Configuration**: Specify entry points in `setup.cfg` as:

  ```ini
  [options.entry_points]
  console_scripts =
      my-app = my_package.__main__:main
  ```
- **Main Function**: The `if __name__ == "__main__":` check should only call a non-async `main()` function, which launches async components if needed. This ensures co

  ```python
  async def run():
      pass

  def main():
      asyncio.run(run())

  if __name__ == "__main__":
      main()
  ```

---

## Performance Profiling

When it comes to Python projects, testing performance can be just as crucial as ensuring functional correctness, especially for resource-intensive applications. By understanding where an application consumes CPU and memory resources, you can optimize and streamline your code, improve user experience, and reduce operational costs.

### Why Performance Profiling Matters

Performance profiling helps detect inefficiencies in the code, uncovering functions or modules that consume disproportionate amounts of CPU or memory resources. By regularly profiling your Python applications or libraries, you can:
1. **Enhance Efficiency:** Eliminate bottlenecks and improve speed and response times.
2. **Optimize Resource Usage:** Reduce unnecessary memory usage and avoid CPU-intensive processes.
3. **Improve Scalability:** Ensure the code can handle higher loads or user requests.
4. **Support Cost-Effectiveness:** Efficient code may require fewer computational resources, lowering infrastructure expenses.

### Types of Performance Profiling

The two primary categories of profiling are **CPU profiling** and **memory profiling**. Let’s explore each in detail.

#### 1. CPU Profiling

CPU profiling measures the time spent executing each part of the code. By identifying which functions are CPU-intensive, you can target specific areas for optimization.

##### Tools for CPU Profiling

1. **cProfile**:
   - **Use Case**: In-built profiler in Python, suitable for a general-purpose overview of where CPU time is spent.
   - **Method**: It provides an output that lists each function call, how often it was called, and how much time it took on average. You can run `cProfile` directly from the command line or embed it within your Python script.
   - **Example Usage**:
     ```python
     import cProfile
     import pstats

     def your_function():
         # Code to profile
         pass

     cProfile.run('your_function()', 'profile_output')
     p = pstats.Stats('profile_output')
     p.strip_dirs().sort_stats('time').print_stats(10)
     ```
   - **Strengths**: Simple to use and well-suited for profiling individual scripts.

2. **pyinstrument**:
   - **Use Case**: Ideal for analyzing high-level performance issues, showing a time breakdown for each function call.
   - **Method**: It generates a flame graph that displays the flow and duration of code execution, making it easier to spot functions with significant overhead.
   - **Example Usage**:
     ```python
     from pyinstrument import Profiler

     profiler = Profiler()
     profiler.start()
     # Code you want to profile
     profiler.stop()
     print(profiler.output_text(unicode=True, color=True))
     ```
   - **Strengths**: Provides an intuitive, easy-to-read breakdown and shows the call stack in a simplified visual format.

3. **line_profiler**:
   - **Use Case**: Focuses on profiling at the line level, showing which lines of code within a function are taking the most time.
   - **Method**: Install via `pip install line_profiler`, then decorate functions with `@profile` to inspect performance at a finer level.
   - **Example Usage**:
     ```python
     @profile
     def your_function():
         # Code to profile line-by-line
         pass
     ```

##### Tips for CPU Profiling

- **Target Specific Code Blocks**: Instead of profiling the entire application, focus on individual functions or methods that are computationally intensive.
- **Profile in Realistic Scenarios**: Run the profiler under conditions that mimic typical usage, as the results will be more applicable for optimizations.
- **Use Flame Graphs**: These visually represent CPU time and can quickly reveal functions with the highest execution costs.

#### 2. Memory Profiling

Memory profiling identifies parts of the code consuming excessive memory, which is essential for applications that handle large datasets or perform in-memory computations.

##### Tools for Memory Profiling

1. **memory_profiler**:
   - **Use Case**: Great for analyzing memory consumption line-by-line, providing detailed insights into memory usage within specific functions.
   - **Method**: Install via `pip install memory-profiler`, then decorate functions with `@profile` to get memory usage breakdowns.
   - **Example Usage**:
     ```python
     from memory_profiler import profile

     @profile
     def your_function():
         # Code you want to profile for memory
         pass
     ```
   - **Strengths**: Line-by-line analysis makes it easy to identify where memory spikes occur.

2. **objgraph**:
   - **Use Case**: Useful for tracking memory leaks by identifying objects that increase memory usage unnecessarily.
   - **Method**: Visualizes object references, making it easy to track the root causes of memory consumption issues.
   - **Example Usage**:
     ```python
     import objgraph

     objgraph.show_growth()
     # Run your function
     objgraph.show_most_common_types()
     ```
   - **Strengths**: Helps to identify memory leaks by revealing object types that are increasing unexpectedly.

3. **heapy (part of Guppy3)**:
   - **Use Case**: Offers in-depth memory analysis by showing the allocation of objects in memory, helping to find memory-intensive objects.
   - **Method**: `heapy` allows you to create snapshots of memory state to compare allocations over time.
   - **Example Usage**:
     ```python
     from guppy import hpy

     h = hpy()
     h.heap()  # Initial snapshot
     # Run your code
     h.heap()  # Post snapshot
     ```
   - **Strengths**: Enables comparisons of memory usage snapshots to detect leaks or inefficient memory allocations.

##### Tips for Memory Profiling

- **Regular Snapshots**: Take memory snapshots at various stages in your code to see how memory usage changes over time.
- **Monitor Object Growth**: Use tools like `objgraph` to observe if certain objects persist or grow in size unexpectedly, indicating potential memory leaks.
- **Optimize Data Structures**: Consider replacing memory-intensive data structures (like lists of dictionaries) with more efficient alternatives (such as Pandas DataFrames or arrays).

#### Best Practices for Performance Profiling

1. **Set Up Automated Profiling**: Integrate profiling into CI/CD pipelines for regular feedback on performance impacts of code changes.
2. **Analyze Results and Act**: After profiling, consider restructuring code, optimizing algorithms, or using efficient libraries to reduce resource usage.
3. **Document and Track**: Keep records of profiling data, especially for frequently executed functions, so you can track improvements or regressions over time.
4. **Optimize Iteratively**: Make incremental improvements rather than overhauling large parts of code at once, which makes it easier to pinpoint gains and losses.

#### Example Workflow for Profiling

To help you get started, here’s a sample workflow:

1. **Identify Hotspots**: Begin with `cProfile` or `pyinstrument` for high-level CPU analysis.
2. **Drill Down**: Use `line_profiler` for specific, time-intensive functions identified in the previous step.
3. **Memory Analysis**: Run `memory_profiler` and `objgraph` on memory-intensive sections to locate excessive memory usage.
4. **Continuous Improvement**: Regularly revisit performance as code evolves, using profiling to guide optimizations for new features or updates.