To install **PostgreSQL** and **pgAdmin** on a Mac using **Homebrew**, you can use the following steps.

-----

### 1\. Install Homebrew (if you haven't already)

Homebrew is a package manager for macOS that makes installing software easy. You can install it by opening the **Terminal** app (found in `/Applications/Utilities`) and running the following command:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

This script will prompt you for your user password. Enter it and press **Enter**. Wait for the installation to complete.

### 2\. Install PostgreSQL

Once Homebrew is installed, you can install PostgreSQL with a single command. In your **Terminal**, run:

```bash
brew install postgresql
```

This command downloads and installs the latest stable version of PostgreSQL, along with its dependencies.

After the installation is finished, you can verify that PostgreSQL is installed correctly by checking its version:

```bash
postgres --version
```

This should output the version number, like `postgres (PostgreSQL) 15.4`.

### 3\. Start PostgreSQL

To start the PostgreSQL server, run the following command. This sets up the server to start automatically when your Mac boots.

```bash
brew services start postgresql
```

To stop the server at any time, you can use `brew services stop postgresql`.

### 4\. Install pgAdmin

**pgAdmin** is a graphical user interface (GUI) for managing your PostgreSQL databases. To install it, you'll use the `brew` command again, but with the `--cask` option, which is used for installing macOS applications (GUIs) instead of command-line tools.

```bash
brew install --cask pgadmin4
```

After the installation is complete, you can find **pgAdmin 4** in your `/Applications` folder.

### 5\. Launch pgAdmin and Connect to PostgreSQL

Launch **pgAdmin** from your `/Applications` folder.

When you open it, you'll be prompted to set a master password. This password is for pgAdmin itself, not for your PostgreSQL database. After setting the password, you'll be taken to the dashboard.

To connect to your local PostgreSQL server:

  * Right-click on **Servers** in the left-hand panel and select **Create \> Server...**.
  * In the **General** tab, give your server a name (e.g., `Local PostgreSQL`).
  * Go to the **Connection** tab.
      * **Host name/address:** `localhost`
      * **Port:** `5432` (the default for PostgreSQL)
      * **Maintenance database:** `postgres`
      * **Username:** The default username for a new Homebrew installation is your macOS username.
      * **Password:** There is no password by default. Leave this field blank.
  * Click **Save**. You are now connected to your local PostgreSQL server and can start creating databases.