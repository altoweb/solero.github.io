---
title: Windows Legacy Setup
layout: tutorial
permalink: /tutorial/legacy/windows
menu:
  introduction:
    name: "Introduction"
    requirements: "Requirements"
  setup:
    name: "Setup"
    install-xampp: "Install XAMPP"
    install-redis: "Install Redis"
    install-python: "Install Python"
    setup-media-server: "Setup media server"
    install-houdini: "Install Houdini"
    start-apache-and-mysql-server: "Start Apache & MySQL service"
    setup-the-database: "Setup the database"
    running-houdini: "Running Houdini"
  testing:
    name: "Testing"
    testing-houdini: "Testing Houdini"
---

# Setup legacy private server on windows
## Introduction

Welcome to the setup guide for legacy private servers. This guide is for the windows operating system. This guide will explain local setup for windows, that means nobody except you will able to play. If you wish to create a private server for everybody to play, please refer to our [Ubuntu setup guide](/tutorial/legacy/ubuntu){:target="_blank"}.

#### Requirements

- A windows computer with at least 1GB of RAM
    - This tutorial is specifically for Windows 10, but the process is the same for all versions of Windows.
- An internet connection
- Ability to breathe

## Setup

### Install XAMPP

First, you will need to install XAMPP, this is an installer for Apache, PHP and MariaDB, which are all required by Houdini. This process is very simple, download the installer and execute it.

Go to the [Apache Friends homepage](https://www.apachefriends.org/index.html){:target="_blank"} and click "XAMPP for Windows" button to download the latest version.

![Windows XAMPP Download](/assets/images/windows/windows_xampp_download.png)

Click "Run" and the installer should execute.

{% capture message %}
If you get a message about User Account Control (UAC) being activated, just click "OK" to continue.
{% endcapture %}
{% include message.html type="info" content=message %}

When the installer starts, click "Next" to move to the "Select Components" section of the installer.

![Windows XAMPP Components](/assets/images/windows/windows_xampp_components.png)

You will see that all of the components are checked, you may continue with all of them, but the only components required for this tutorial are **Apache, PHP, MySQL & PHPMyAdmin**. You may uncheck the rest of the components as they are not required.

You may now click "Next" until the installer starts. 

![Windows XAMPP Install](/assets/images/windows/windows_xampp_install.png)

Finally, click "Finish" to exit the installer and open the XAMPP control panel. You may minimize the XAMPP control panel as you do not need it for now.

### Install Redis

Redis is required to use Houdini. Installing Redis is very easy, just download the [installer](https://github.com/MicrosoftArchive/redis/releases){:target="_blank"} and execute it. 

![Windows Redis install](/assets/images/windows/windows_redis_install.png)

{% capture message %}
When asked if you wish to add the Redis installation folder to the PATH environment variable, you may want to check this as it can make debugging issues easier later on.
{% endcapture %}
{% include message.html type="info" content=message %}

If you require a 32-bit Windows installer for Redis, you can find them [here](https://github.com/rgl/redis/downloads){:target="_blank"}.

Leave all other settings in the installer the same, just click "Next" until the installation starts. Finally, click "Finish" to exit the installer.

### Install Python

In order to run Houdini, you will need to install Python 2.7. This is very easy, just download the [MSI installer](https://www.python.org/downloads/){:target="_blank"} and execute it.

{% capture message %}
Do **not** install Python 3.x! It will not work with Houdini as it does not yet support Python 3. You must use Python 2.7. At the time of writing this, the latest version of Python 2 is Python [2.7.15](https://www.python.org/downloads/release/python-2715/){:target="_blank"}.
{% endcapture %}
{% include message.html type="warning" content=message %}

![Windows Python download](/assets/images/windows/windows_python_download.png)

When the install starts, click "Next" until you get to the "Customize Python" section of the installer. Scroll to the bottom of the list and find "Add python.exe to PATH", click the drop down menu to the left and change the selected option to "Will be installed on local hard drive".

![Windows Python install](/assets/images/windows/windows_python_install.png)

Finally, click "Next" and wait for the installer to finish, click "Finish" to exit the installer.

### Setup media server

You can download a pre-configured media server from the Icerink archive [here](https://icer.ink/.repo/legacy/media.zip){:target="_blank"}.

It may take a while to download depending on your internet connection. 

Whilst the media is downloading, you will need to go and configure local subdomains for Apache. You may already have the XAMPP control panel open, if not, open the Windows start menu and search "XAMPP Control Panel", to bring up the XAMPP control panel.

![Apache config](/assets/images/windows/apache_config.png)

Next, go to the control panel, click the "Config" button next to "Apache" and go to "&lt;Browse&gt; [Apache]". Go to folder `conf` then `extra` and open file `httpd-vhosts.conf` with Notepad. 

![Apache vhosts](/assets/images/windows/apache_vhosts.png)

Paste the following at the bottom of the file.

```conf
<VirtualHost *:80>
    ServerAdmin webmaster@play.localhost
    DocumentRoot "C:/xampp/htdocs/play"
    ServerName play.localhost
    ErrorLog "logs/play.localhost-error.log"
    CustomLog "logs/play.localhost-access.log" common
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin webmaster@media.localhost
    DocumentRoot "C:/xampp/htdocs/media"
    ServerName media.localhost
    ErrorLog "logs/media.localhost-error.log"
    CustomLog "logs/media.localhost-access.log" common
</VirtualHost>
```

Save the file and close Notepad.

![Apache vhosts save](/assets/images/windows/apache_vhosts_save.png)

Go to the Windows start menu and search for "Notepad", right click the Notepad application and select "Run as administrator".

![Windows notepad search](/assets/images/windows/windows_notepad_search.png)

In notepad, click "File" and then "Open" (or just press `Ctrl + O`). You should see an explorer window open at `C:/Windows/System32`, inside this folder, go to `drivers` and then `etc`. Double click on file `hosts` to open it.

![Notepad hosts](/assets/images/windows/notepad_hosts.png)

{% capture message %}
If you do not see any files in the explorer window after navigating to `drivers` and then `etc`, you probably need to change file type in windows explorer from "Text Documents" to "All Files".

![Notepad file type](/assets/images/windows/notepad_file_type.png)
{% endcapture %}
{% include message.html type="warning" content=message %}

Paste the following at the bottom of the file.

```
127.0.0.1	play.localhost
127.0.0.1	media.localhost
```

Save the file and close Notepad.

![Hosts save](/assets/images/windows/hosts_save.png)

Now, go back to the XAMPP control panel and click the "Explorer" button. 

![XAMPP Explorer](/assets/images/windows/xampp_explorer.png)

Now go to the folder `htdocs` and delete **all** of the contents. You can drag-highlight them all, right click and then "Delete" or `Ctrl + A`, right click and then "Delete".

![htdocs delete](/assets/images/windows/htdocs_delete.png)

Your `htdocs` folder should now be empty.

Open a new explorer window and navigate to where you saved your Icerink media server you downloaded at the beginning of this step. Double click the `media.zip` file you downloaded. 

Highlight the two folders inside `play` and `media` and drag them over to the `htdocs` explorer window.

![extract media](/assets/images/windows/extract_media.png)

The files will now extract, this may take a short while, you may continue with the tutorial whilst this completes, you are finished with this step.

### Install Houdini

Now you will need to setup Houdini, this is the super-special magic sauce that allows your Club Penguin private server to function! Go to the [Houdini GitHub repository](https://github.com/Solero/Houdini){:target="_blank"} and click "Clone or download", then click "Download ZIP" to download Houdini as a ZIP archive.

{% capture message %}
If you have [git](https://git-scm.com/download/win){:target="_blank"} installed on your Windows computer, you may open a command prompt and clone the repository.

`git clone https://github.com/solero/houdini`

If not, ignore this box!
{% endcapture %}
{% include message.html type="info" content=message %}

![Houdini save](/assets/images/windows/houdini_save.png)

You may save Houdini wherever you like, for this tutorial, Houdini will be saved on the windows desktop.

Go to where you saved the Houdini ZIP archive from GitHub, and extract it.

![extract houdini](/assets/images/windows/houdini_extract.png)

Open the folder you just extracted. Inside should be a folder which looks like this.

![houdini folder](/assets/images/windows/houdini_folder.png)

Hold down `Shift` on your keyboard and right click in the explorer window displaying the files shown above, in the context menu, click on "Open command window here".

![open command prompt](/assets/images/windows/command_prompt.png)

{% capture message %}
On newer versions of Windows 10, you may see "Open PowerShell here" instead of "Open command window here", this is fine, PowerShell can act as a replacement for command prompt.

**Make sure you are holding `Shift` when you right click!** Otherwise you will not see the option.
{% endcapture %}
{% include message.html type="warning" content=message %}

When the command prompt (or PowerShell) window opens up, enter the following command and press enter.

```
pip install -r requirements.txt
```

You should see all of Houdini's requirements install.

![requirements install](/assets/images/windows/requirements_install.png)

At the end you should see a message similar to this.

```
Successfully installed Automat-0.7.0 PyHamcrest-1.9.0 PyYAML-3.13 SQLAlchemy-1.2.2 Twisted-18.7.0 ...
```

You may also see messages about the pip version being out of date, this is fine. If you see any errors (commonly shown in red) then you may need to [ask for help on the forums](https://solero.me){:target="_blank"}.



### Start Apache and MySQL service

Now you will need to start both the Apache and MySQL service. Go back to the XAMPP control panel.

![Windows XAMPP control](/assets/images/windows/windows_xampp_control.png)

Click "Start" next to both Apache and MySQL. Afterwards, the control panel should look similar to above (Apache & MySQL both in green highlight). Finally, you may now close the XAMPP control panel.

### Setup the database

Open a web browser and navigate to `http://localhost/phpmyadmin`. You should see the PHPMyAdmin interface.

Click "New" above the list of databases.

![phpmyadmin](/assets/images/windows/new_database.png)

Enter "Houdini" in "Database name" entry field and click "Create".

![create database](/assets/images/windows/create_database.png)

Now click "Import" along the top pane and then "Browse".

![database browse](/assets/images/windows/database_browse.png)

An explorer window should appear, browse to where you extracted Houdini, and find the `houdini.sql` file inside, select that file and click "Open".

![sql install](/assets/images/windows/sql_install.png)

Scroll to the bottom of the page and click the "Go" button. You should see a message stating that the SQL file has imported correctly.

![import success](/assets/images/windows/import_success.png)

### Running Houdini

Open the folder where you extracted Houdini in a new explorer window. Hold shift and right click, then click "Open command window here" (or PowerShell).

A command window should come up. Enter the following command and press enter.

```
python Login.py
```

You should see output similar to below.

![running houdini](/assets/images/windows/houdini_run.png)

If you see anything else, something has gone wrong. You can ask for help on the [forums](https://solero.me){:target="_blank"}.

Minimise this window, and repeat this step (Open another command window).

This time, run the following commmand.

```
python World.py
```

You should see output similar to below.

![running houdini world](/assets/images/windows/houdini_run_world.png)

You should now have two command windows open running both `Login.py` and `World.py`.

## Testing

### Testing Houdini

Congrats! You're done. Just navigate to `http://play.localhost` in your browser and sign in with the test account!

Username: `Basil`

Password: `password`

![testing](/assets/images/windows/testing.png)