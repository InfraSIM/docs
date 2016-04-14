# About

This repo is the InfraSIM documentation's source code ( http://infrasim.readthedocs.org )

[![Documentation Build Status](https://readthedocs.org/projects/infrasim/badge/?version=latest)](https://readthedocs.org/projects/infrasim/?badge=latest)

- if you are a user of InfraSIM and would like to find documents, Please go to:  http://infrasim.readthedocs.org/
- if you are going to contribute the the InfraSIM's documents, you can refer to follow below steps , which explain how to edit and build the documentation on a local system.

## Steps for Linux

1. Create a virtualenv and set up the requirements.

   ```    
   virtualenv .venv
   source .venv/bin/activate
   cd docs
   pip install -r requirements.txt
   ```

2. Build the HTML documentation in the *_build* directory.

   ```
    cd docs
   make html
   ```

3. To automatically rebuild the documentation as you update the documentation files.

    ```
    cd docs
   sphinx-autobuild -H 0.0.0.0 -p 8000 . _build/html
    ```

   Access the documentation by navigating to http://127.0.0.1:8000.

## Steps for Windows

**Prerequisites:** Ensure that Sphinx, Python, and a Sphinx-supported editor is installed on the system. (The project team uses Atom as the editor.) For Pythin, install https://www.python.org/ftp/python/2.7.0/python-2.7.10.msi with the default location and components.


1. Clone the *docs* repository from Github to your system.

2. Install and set up the virtualenv

   ```
   c:\python27\scripts\pip install virtualenv
   c:\python27\scripts\virtualenv .venv
   .venv\scripts\activate
   pip install -r requirements.txt
   ```

3. Build the HTML documentation.

   ```
   venv\scripts\activate
   cd docs
   .\make.bat html
   ```

4. Access the documentation in a browser.

   ```
   .venv\scripts\activate
   cd docs
   start .\_build\html\index.html
   ```

5. Preview the changes , by automatically rebuilding the documentation as you make changes.

   ```
   sphinx-autobuild -H 0.0.0.0 -p 8000 . _build/html
   ```

   Access the documentation by navigating to http://127.0.0.1:8000

## Push change to online docs

   you don't need to do anything. After your Pull Request gets merged into "docs" repository, CI(Continous Intergration) will push and update the online docs(http://infrasim.readthedocs.org) for you .
