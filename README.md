# uvtarget

**uvtarget is a helpful utility to manage Python in CMake, powered by uv**

### Features
* Supports autognerating `pyproject.toml` for a workspace
* Creates a dev workspace with all declared projects included as editable
* Automatically syncs changes to projects to the workspace, every build (possible because `uv` is *fast*)
* Supports installation to a target virtual env
* Supports `FindPython` either as a preceeding step to point `uvtarget` at an existing Python version, or as a successive step to allow linking your code against a uv installed Python.

### Why do I want this?
* `uv lock` in a repo with multiple packages requires a single top level package to bring together all other projects. 
    * At least starting off, it can be helpful for some of the boilerplate to be taken away (`uvtarget` doesn't require project generation but does encourage it)
    * Especially for container workflows, it's important not to have multiple versions of the same library floating around, and to be able to pin an entire repo at once
* Simplify dev workflows with CMake
    * I need `jinja2`, how am I going to install it? `apt`? `pip`? What if the version I need isn't compatible with my environment? How do I ensure that users of my CMake extension or library have the right versions installed?
    * What if I want to build my code against a different python version? What if that version isn't in the system package manager?
    * I want to make sure that 
* Properly handle `sudo make install`
    * By default this doesn't work well with `uv` - both in terms of the root user finding the binary as well as the venv being set up pointed to a python binary that's accessible to the regular user
    * The steps required to take a project+lock file and install it to another location aren't terribly complicated, but they also aren't straightforward to guess. It's nice to not have to dig through blogs/forum posts to find them.
* Properly sandbox away the environment details from the user's terminal from the build
    * If you have a current sourced virtual environment, it can affect certain `uv` invocations, either targeting them at the current environment or project, or printing a warning asking you if you meant to do so.
* Handle workspace members or other dependencies that might depend on the value of some CMake variable
    * This is likely doable with clever use of dependency-groups, but it's nice to let CMake drive it, especially if you don't know the package names up front.

None of these are impossible without something like `uvtarget`, but it sure does help.

### Need something this doesn't provide?

You're welcome to make a feature request or PR. Beyond that - if it's some addition to the generated `pyproject.toml`, I'd recommend swapping to `UNMANAGED`, if the feature is complex. If it's just some custom flag to `uv sync` or change in the install feature, I recommend you fork and change it yourself if things are moving too slowly.

### Learn even more
See this blog post

### Example Usage
```cmake
# Early in your top level CMakeLists.txt

set(MY_PYTHON_VERSION 3.12)

uv_initialize(
    # Pin to a specific Python verison
    PYTHON_VERSION ${MY_PYTHON_VERSION}
    # Move the workspace pyproject+lock to a subdirectory
    MANAGED_PYPROJECT_FILE python/pyproject.toml
    # Give the workspace a name
    WORKSPACE_PACKAGE_NAME basis_cmake
    # Enable installation, to a venv at this path
    INSTALLATION_VENV /opt/basis/.venv
    # Setup storage for the venv, required for `sudo make install`
    INSTALLATION_VENV_CACHE /opt/basis/cache
    )
```

```cmake
# my_awesome_lib/CMakeLists.txt

# Add a project to the workspace
# Will both install as editable along with dependencies
uv_add_pyproject(pyproject.toml)
```

```cmake
# cmake/jinjafy.cmake

# Add a tool that's needed by a CMake target
uv_add_dev_dependency("jinja2>=2.0.0")

# Call a python script
add_custom_command(
    COMMAND
        # Will use the jinja version declared above
        ${UV} run ${BASIS_SOURCE_ROOT}/do_template_magic.py 
    ...
    )
```

```cmake
# links_against_py/CMakeLists.txt

# Correctly finds the Python3.12 installed by uv
find_package(Python ${MY_PYTHON_VERSION} REQUIRED COMPONENTS Development Interpreter)

add_executable(links_against_py main.cpp)
target_link_libraries(links_against_py PUBLIC Python::Python)
```

### Installation

Either use FetchContent or insource this into your own repo.

```cmake
TODO once this is moved out

```