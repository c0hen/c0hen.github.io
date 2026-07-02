---
layout: default
title: Python tips
description: Python-venv usage to install applications with all dependencies and assorted tips
tags: python beginner tips programming system distribution packaging dependencies uv build
---

## Debug with the [python debugger](https://docs.python.org/3/library/pdb.html)

In code
```python
breakpoint()
```

Command line interface
```sh
python -m pdb --help
```

## Recover a broken python installation (errors like: pycompile not found)

Might need to reinstall only python-minimal, reinstall other packages as needed.

```
apt-get install --reinstall python python-minimal python-setuptools
```

## Guard scripts against accidental invocation

Option to protect against accidental invocation on import of a file. Better to write clean modules to import but this way is common in python.

```python
#!/usr/bin/env python

def is_tool(name):
  """Check if `name` is on PATH and is executable."""
  from shutil import which
  return which(name) is not None

if __name__ == '__main__':
  """Main code protected from accidental invocation."""
  tool = "ping"
  if is_tool(tool):
    print("We have ping")
```

A script imported without this `if __name__ == '__main__':` guard block will be triggered by the importing script:

- to run at import time
- using the importing script's command line arguments.

This is almost always a mistake.

If there is a custom class in the guardless script and it is saved to a pickle file, then unpickling it in another script will trigger an import of the guardless script, with the same problems.

## Debian: use alternatives to set python version

1 is the lowest priority (last group on the line).

```
#update-alternatives --install /usr/bin/python python /usr/bin/python3 2
#update-alternatives --install /usr/bin/python python /usr/bin/python2 1
```

## Install applications using venv (virtual environment)

Install

```
aptitude install python3 python3-pip python3-venv

```

Creation of the virtual environment:

```
python3 -m venv django-venv
```

*Getting an error like*

```
ImportError: cannot import name 'Popen' from partially initialized module 'subprocess' (most likely due to a circular import)
```

*could mean you have a file named subprocess.py in the directory you're in. Any python module names being used as a file name will probably trigger this. It'll make python try to override its module and find the "module" in your file is not a module.*

Create and end the session you'll be installing your app in:

```
source bin/activate
(django-venv)deactivate
```

Preparation in the session:

```
python3 -m pip install --upgrade pip
```

Install your app:

```
python3 -m pip install django
```

or 

```
python3 -m pip install django~=1.10.4
```

for version matching.

## Directly from python packaging guide

[https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/)

### Installing extras

Some packages have optional extras. You can tell pip to install these by specifying the extra in brackets:

```
python3 -m pip install requests[security]
```

### Installing from source

python3 -m pip can install a package directly from source, for example:

```
cd google-auth
python3 -m pip install .
```

Additionally, pip can install packages from source in development mode, meaning that changes to the source directory will immediately affect the installed package without needing to re-install:

```
python3 -m pip install --editable .
```

### Installing from version control systems

python3 -m pip can install packages directly from their version control system. For example, you can install directly from a git repository:

```
git+https://github.com/GoogleCloudPlatform/google-auth-library-python.git#egg=google-auth
```

For more information on supported version control systems and syntax, see pip’s documentation on VCS Support.

### Installing from local archives

If you have a local copy of a Distribution Package’s archive (a zip, wheel, or tar file) you can install it directly with pip:

```
python3 -m pip install requests-2.18.4.tar.gz
```

If you have a directory containing archives of multiple packages, you can tell pip to look for packages there and not to use the Python Package Index (PyPI) at all:

```
python3 -m pip install --no-index --find-links=/local/dir/ requests
```

This is useful if you are installing packages on a system with limited connectivity or if you want to strictly control the origin of distribution packages.

### Using other package indexes

If you want to download packages from a different index than the Python Package Index (PyPI), you can use the --index-url flag:

```
python3 -m pip install --index-url http://index.example.com/simple/ SomeProject
```

If you want to allow packages from both the Python Package Index (PyPI) and a separate index, you can use the --extra-index-url flag instead:

```
python3 -m pip install --extra-index-url http://index.example.com/simple/ SomeProject
```

### Using requirements files

Instead of installing packages individually, pip allows you to declare all dependencies in a Requirements File. For example you could create a requirements.txt file containing:

```
requests==2.18.4
google-auth==1.1.0
```

And tell pip too install all of the packages in this file using the -r flag:

```
python3 -m pip install -r requirements.txt
```

#### Constraints files

Constraints files are in the format of requirements. They control which version of a package can be installed but provide no control over the actual installation. For example, a production environment can have hardened versions of packages specified via constraints.

```sh
python -m pip -c constraints.txt
PIP_CONSTRAINTS='https://example.com/constraints.txt' python -m pip
```

### Freezing dependencies

Pip can export a list of all installed packages and their versions using the freeze command:

```
python3 -m pip freeze
```

Which will output a list of package specifiers such as:

```
cachetools==2.0.1
certifi==2017.7.27.1
chardet==3.0.4
google-auth==1.1.1
idna==2.6
pyasn1==0.3.6
pyasn1-modules==0.1.4
requests==2.18.4
rsa==3.4.2
six==1.11.0
urllib3==1.22
```

This is useful for creating Requirements Files that can re-create the exact versions of all packages installed in an environment.

### Filesystem - use glob to delete backups without regex

```python
#!/usr/bin/env python3
# %z not supported in python2
# Delete backup files by entity, determine entity from the file name. Successful backups must be larger than size, smaller are not counted. At least 3 successful backups kept (warning if first older than 35 days), no more than 3 if older than 21 days.

import glob, os
import time
import datetime

last = ""   # entity's last backup
count = 0   # entity's count of backups
cur_timestamp = time.time()
timeformat = "%Y%m%d%H%M%S%z"

for infile in sorted(glob.glob('./*.tar.gz'), reverse=True):
    filename = infile[:-7]
    entity = filename[:-19]
    date1 = filename[-19:]
    
    file_info = os.stat(infile)
    size = file_info.st_size
    
    if (entity != last):    # new entity
        count = 0
        last = entity
        print( str(entity) + ":----------" ) 
    
    if (size > 50000): count+=1 
    
    timestamp = time.mktime(datetime.datetime.strptime(date1, timeformat).timetuple())
    age = int((cur_timestamp-timestamp)//60/60/24) # file's age in days
    
    print( str(count) + ": " + entity + " " + date1 + " size: " + str(size) +  " age: "+ str(age) + " d." )
    
    if (age > 21 and count > 3):
        print( "rm: count: " + str(count) + ", age " + str(age) + ": " + infile)
        os.remove(infile)
    elif (age > 35):
        print( str(count) + ": " + str(age) + " days old, " + entity + " " + date1 + "   " + infile )
```

### Document, test

Debugging is enabled by default in python.
A docstring is the first line of a module, function, class, or method definition.
Docstrings, formatted like git commit messages, are removed when using `python -OO`
```python
def printTruth():
  r"""This method prints truth.

  This is a raw docstring in case there's a \
  in the docstring."""
  print("True")
```
```python
print(printTruth.__doc__)
```
An [assert](https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement) statement and any code conditional on the value of `__debug__` is removed with `python -O`
```python
assert age == 2
```
is equivalent to
```python
if __debug__:
    if not age == 2: raise AssertionError
```

[Pytest](https://docs.pytest.org/en/stable)

### Python string manipulation

#### Appending or extending a list

Append when adding a single element, extend when adding all elements of a list.

```python
result = [
  "This list will contain",
]
s = "all elements of the list a: "
a = [
  "fox",
  "wolf",
  "bear",
]
result.append(s)
result.extend(a)
print(result)
```

## Creating packages

Old package format used by setuptools was egg. Left from that era is the directory name suffix `.egg-info`.
```bash
example_project.egg-info/
├── dependency_links.txt
├── entry_points.txt
├── PKG-INFO
├── requires.txt
├── SOURCES.txt
└── top_level.txt
```
Current package format is wheel `.whl`.

### Package structure

Modern packages are [declared in](https://packaging.python.org/en/latest/specifications/pyproject-toml/#pyproject-toml-spec) `pyproject.toml`. This allows selecting a build system and replaces `setup.py`.

Historical `setuptools` convention dictated using `__init__.py` to mark a python package. There were 3 schools of thought.

1. Leave the `__init__.py` blank. This enforces explicit imports and thus clear namespaces.
1. Import all modules in __init__.py. The user does not have to do multiple imports but no API stability.
1. Import key functions from various modules directly into the package namespace. If modules are restructured, API can be kept the same for end users.

Minimal `setup.py` that can be used with `pyproject.toml` for compatibility.
```
import setuptools

setuptools.setup()
```

Sample repository structure using `uv`.

```bash
├── docs
│   └── sample.md
├── LICENSE
├── pyproject.toml
├── .python-version
├── README.md
├── requirements.txt
├── src
│   └── sample
│       ├── __init__.py
│       ├── sample.py
│       └── helpers.py
├── tests
│   └── test.py
└── uv.lock
```

### Automatic documentation from docstrings using [pdoc](https://pdoc.dev/docs/pdoc.html)

Supports Jinja2 templates to customize output. Main use case for pdoc is API documentation, more complex needs are better served by Sphinx.

```sh
pdoc --help
pdoc ./demo.py -o ./docs
```

Variables are not assigned the `__doc__` attribute by python. `pdoc` will read the abstract syntax tree (an abstract representation of the source code) and include all assignment statements immediately followed by a docstring.

The public interface (API) of a module is determined through one of two ways.

- If `__all__` is defined in the module, then all identifiers in that list will be considered public. No other identifiers will be considered public.
- If `__all__` is not defined, then pdoc will consider all items public that do not start with an underscore and that are defined in the current module (i.e. they are not imported).

If you want to override the default behavior for a particular item, you can do so by including an annotation in its docstring:

- `@private` hides an item unconditionally.
- `@public` shows an item unconditionally.

#### Sphinx autodoc

Pdoc is meant to be compatible with the autodoc extension for Sphinx.

```
# docs/conf.py
extensions = [
  ...
  'sphinx.ext.autodoc',
  'myst_parser', # markdown support for sphinx
]

```
[Info fields](https://www.sphinx-doc.org/en/master/usage/domains/python.html#info-field-lists) can be added to docstrings for Sphinx.
```python
def my_function(*args):
  """Example function

  :meta private:
  """
```

### [uv to manage project](https://docs.astral.sh/uv/guides/) instead of venv and pip

```sh
uv init hello
uv version --bump patch --bump dev --dry-run
uv lock --upgrade-package requests
uv help venv
uv build
```

Initialize a package, moves code to `src/` directory and defines a build system.
```sh
uv init --package hello
```

#### Python script that will create its environment with uv

Inline script metadata according to PEP 723.

```
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.13"
# dependencies = [
#     "biopython",
# ]
# ///
```

#### Work interchangeably with pip in other parts of the pipeline.

```sh
uv export --no-hashes --format requirements-txt --output-file requirements.txt
uv add -r requirements.txt
```

#### Create executable scripts in the package

Same process as in [python packaging guide](https://packaging.python.org/en/latest/guides/writing-pyproject-toml), using an entry point to create `spam-cli`.
```
# pyproject.toml
[project.scripts]
spam-cli = "spam:main_cli"
```
effectively does
```python
import sys
from spam import main_cli
sys.exit(main_cli())
```
