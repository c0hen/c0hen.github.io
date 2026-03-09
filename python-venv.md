---
layout: default
title: Python tips
description: Python-venv usage to install applications with all dependencies and assorted tips
tags: python beginner tips programming system distribution
---

## Recover a broken python installation (errors like: pycompile not found)

Might need to reinstall only python-minimal, reinstall other packages as needed.

```
apt-get install --reinstall python python-minimal python-setuptools
```

## Debian: use alternatives to set python version

1 is the lowest priority (last group on the line).

```
#update-alternatives --install /usr/bin/python python /usr/bin/python3 2
#update-alternatives --install /usr/bin/python python /usr/bin/python2 1
```

## Install applications using virtualenv

Install 

```
aptitude install python3
python3 -m pip install --user --upgrade pip
python3 -m pip install --user virtualenv

```

Creation of the virtual environment:

```
python3 -m virtualenv django-venv
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
pip install --upgrade pip
```

Install your app:

```
pip install django
```

or 

```
pip install django~=1.10.4
```

for version matching.

## Directly from python packaging guide

[https://packaging.python.org/guides/installing-using-pip-and-virtualenv/](https://packaging.python.org/guides/installing-using-pip-and-virtualenv/)

### Installing extras

Some packages have optional extras. You can tell pip to install these by specifying the extra in brackets:

```
pip install requests[security]
```

### Installing from source

pip can install a package directly from source, for example:

```
cd google-auth
pip install .
```

Additionally, pip can install packages from source in development mode, meaning that changes to the source directory will immediately affect the installed package without needing to re-install:

```
pip install --editable .
```

### Installing from version control systems

pip can install packages directly from their version control system. For example, you can install directly from a git repository:

```
git+https://github.com/GoogleCloudPlatform/google-auth-library-python.git#egg=google-auth
```

For more information on supported version control systems and syntax, see pip’s documentation on VCS Support.

### Installing from local archives

If you have a local copy of a Distribution Package’s archive (a zip, wheel, or tar file) you can install it directly with pip:

```
pip install requests-2.18.4.tar.gz
```

If you have a directory containing archives of multiple packages, you can tell pip to look for packages there and not to use the Python Package Index (PyPI) at all:

```
pip install --no-index --find-links=/local/dir/ requests
```

This is useful if you are installing packages on a system with limited connectivity or if you want to strictly control the origin of distribution packages.

### Using other package indexes

If you want to download packages from a different index than the Python Package Index (PyPI), you can use the --index-url flag:

```
pip install --index-url http://index.example.com/simple/ SomeProject
```

If you want to allow packages from both the Python Package Index (PyPI) and a separate index, you can use the --extra-index-url flag instead:

```
pip install --extra-index-url http://index.example.com/simple/ SomeProject
```

### Using requirements files

Instead of installing packages individually, pip allows you to declare all dependencies in a Requirements File. For example you could create a requirements.txt file containing:

```
requests==2.18.4
google-auth==1.1.0
```

And tell pip too install all of the packages in this file using the -r flag:

```
pip install -r requirements.txt
```

### Freezing dependencies

Pip can export a list of all installed packages and their versions using the freeze command:

```
pip freeze
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
An [assert](https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement) statement and any code conditional on the value of __debug__ is removed with `python -O`
```python
assert age == 2
```
is equivalent to
```python
if __debug__:
    if not age == 2: raise AssertionError
```

[Pytest](https://docs.pytest.org/en/stable)
