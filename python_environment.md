You can use **`venv`** by following a few simple steps. `venv` is a built-in Python module that lets you create isolated environments for your projects. These environments keep your project's dependencies separate from your system's global Python packages, preventing conflicts and ensuring your code runs consistently.

### 1\. Create a virtual environment

First, navigate to your project directory in your terminal. Then, run the following command to create a new virtual environment. You can name it whatever you like, but **`venv`** or **`.venv`** are common conventions.

```bash
python3 -m venv venv
```

This command creates a directory named **`venv`** (or your chosen name) inside your project folder. This directory contains the necessary files and a copy of the Python interpreter to create an isolated environment.

-----

### 2\. Activate the virtual environment

Before you install any packages, you must activate the virtual environment.

**On macOS and Linux:**

```bash
source venv/bin/activate
```

**On Windows:**

```bash
.\venv\Scripts\activate
```

After activation, you'll see the name of your virtual environment (e.g., `(venv)`) in your terminal's prompt. This indicates that you are now working inside the isolated environment.

-----

### 3\. Install packages

With the environment activated, you can now use `pip` to install any packages your project needs. These packages will be installed only within this environment, not on your system globally.

```bash
pip install requests
```

To see which packages are installed in your environment, run:

```bash
pip list
```

A common practice is to create a **`requirements.txt`** file that lists all your project's dependencies. To generate this file, run:

```bash
pip freeze > requirements.txt
```

To install all packages listed in a **`requirements.txt`** file, run:

```bash
pip install -r requirements.txt
```

-----

### 4\. Deactivate the virtual environment

When you're finished working in your project, simply run the `deactivate` command to exit the virtual environment and return to your system's default Python.

```bash
deactivate
```

The terminal prompt will return to normal, indicating that the virtual environment is no longer active.