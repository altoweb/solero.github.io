---
title: Ubuntu Legacy Setup
layout: tutorial
permalink: /tutorial/legacy/ubuntu
menu:
  introduction:
    name: "Introduction"
    requirements: "Requirements"
  setup:
    name: "Setup"
    install-packages: "Install packages"
    setting-up-the-media-server: "Setting up the media server"
    install-houdini: "Install houdini"
    create-database: "Create database"
    configuring-houdini: "Configuring Houdini"
    configuring-subdomains: "Configuring subdomains"
    running-houdini: "Running Houdini"
    test-your-server: "Test your server"
  post-setup:
    name: "Post setup"
    user-registration: "User registration"
    managing-the-database:
      name: "Managing the database"
      datagrip: "DataGrip"
      phpmyadmin: "PHPMyAdmin"
    managing-houdini:
      name: "Managing Houdini"
      restarting-houdini: "Restarting Houdini"
      installing-plugins: "Installing plugins"
      updating-houdini: "Updating Houdini"
    securing-your-server:
      name: "Securing your server"
  miscellaneous:
    name: "Miscellaneous"
    reccomended-hosting-providers: "Reccomended hosting providers"
    other-useful-resources: "Other useful resources"
---

# Setup legacy AS2 private server
## Introduction
Welcome to the setup guide for legacy AS2 private servers. This guide is for the Ubuntu operating system and it helps if you have basic understanding of a UNIX-like terminal, it definitely **is not** a requirement but it certainly helps. If you have never used Linux before, now is an excellent opportunity to learn! In fact, not only is it a really useful skill for guides such as this, but it is handy information to know when dealing with computers in general.

If you do not have access to a machine running Ubuntu, you may run a virtual machine to test locally or alternatively, follow [our other OS specific guides](/tutorial/legacy){:target="_blank"}.

#### Requirements:
- A server, desktop or virtual machine running Ubuntu 16.04 or higher
  - For best performance this machine should probably have at least **1GB** of memory.
- An internet connection
- A brain

{% capture message %}
If you want to host your private server for everyone to play you are also going to need the following requirements.

- A VPS or dedicated server running Ubuntu 16.04 or higher
  - [See here](#reccomended-hosting-providers) for a list of recommended hosting providers. *Do not use free VPS providers!* *You cannot use normal web hosting to run a public private server!*
- A domain name
  - This is the URL others will go to to find your server (ex: mycpps.com). You can buy them [here](http://namecheap.com){:target="_blank"}.
- An SSH client
  - This is how you will communicate with your server. If you're using windows to connect you can use [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html){:target="_blank"}. MacOS users may use [open ssh](http://accc.uic.edu/answer/how-do-i-use-ssh-and-sftp-mac-os-x){:target="_blank"}, just open the terminal and type `ssh root@vpsiphere`.
{% endcapture %}

{% include message.html type="info" content=message %}

## Setup

### Install packages
Before we begin, it's always a good idea to update your package list before starting to install packages. Update them by running this command.
```shell
sudo apt-get update 
```

The very first step is to install all the required Ubuntu packages. Here is a command that gets all this out the way in a single blow!
```shell
sudo apt-get install nginx python2.7 python-pip python-dev python-setuptools mariadb-server libmariadbclient-dev redis-server 
git unzip wget sed gcc
```

You will likely be asked to enter your password, do not be startled when the password doesn't show as you type, just type the password and press enter. If you are asked if you wish to continue, type `y` and then enter.

{% capture message %}
**Heads up!** if you are going to host a public private server it is **IMPERATIVE** that you set a MySQL database password. You can easily harden your installation by running this command:

`sudo mysql_secure_installation`

When asked to set a root password type `y` and then press enter. Enter your chosen password and enter again to confirm. Then type `y` and enter for every following prompt until the command is finished.
{% endcapture %}

{% include message.html type="danger" content=message %}

### Setting up the media server
In order to create a private server you'll need a media server, this is a collection of the game assets that make up the Club Penguin client.
```shell
wget -q https://skihill.net/.repo/legacy/setup && sudo bash setup
```

This may take a while to complete depending on your machine's connection, sit back and av' a cuppa' while it completes!

After a while the media server setup script will ask you to enter "hostname", if you're making a public private server, then you will enter your domain name here (ex. `iclubpenguin.com`), otherwise just press enter.

The script will next ask you to enter your external IP address, if you are making a public private server, then you will enter your VPS' external IP here (ex. `74.125.224.72`), otherwise just press enter.

### Install Houdini
Now we're going to install Houdini, this process is pretty simple.
```shell
git clone https://github.com/ketnipz/houdini ~/houdini
python -m pip install -r ~/houdini/requirements.txt
```

### Create database
You're going to need a database to store all your penguins, to create one, you're going to have to execute Houdini's SQL file on the MySQL server.

First, let's create a new MySQL database, in this case, named `Houdini`.
```shell
sudo mysql -u root -p -e "create database Houdini";
```

If you're prompted for your password, enter your MySQL database password that you set when you ran `sudo mysql_secure_installation`, and press enter. If you didn't set a MySQL database password, just press enter.

{% capture passwordmsg %}
Occasionally you won't be able to login to MySQL even after setting your password, you may get `Access denied for user 'root'@'localhost' (using password: YES)` error.

If this happens to you, you must stop the MySQL server and start with `--skip-grant-tables` flag to gain access and reset the root password. You can follow [this tutorial](https://stackoverflow.com/questions/20353402/access-denied-for-user-testlocalhost-using-password-yes-except-root-user){:target="_blank"} to learn how to do this.
{% endcapture %}

{% include message.html type="info" content=passwordmsg %}

Now, let's install the `houdini.sql` on the database we've just created. You will likely be asked to enter your MySQL password once again.

```shell
sudo mysql -u root -p Houdini < ~/houdini/houdini.sql
```

### Configuring Houdini

Now we have to configure Houdini, for the purpose of this tutorial, you just need to edit one line, however Houdini actually has lots more configuration options which you may play around with.

```shell
nano ~/houdini/config.py
```

This brings up a terminal text editor, you should see some lines, near the top, which look similar to this.
```python
...
"Database": {
        "Address": "localhost",
        "Username": "root",
        "Password": "",
        "Name": "Houdini",
        "Driver": "PyMySQL" if sys.platform == "win32" else "MySQLdb"
},
...
```
Use your arrow keys to move the cursor inbetween the quotes following password, and type your MySQL password here. The line should now look like this

`"Password": "your_mysql_password_here",`

Save and exit by following these steps:
Hit `Ctrl+X` (`cmd X` on MacOS) and then `y`, then enter to save.

### Configuring subdomains
{% capture message %}
You do not need to follow these steps if you are **not** hosting **publicly**, skip this step if you are hosting locally!
{% endcapture %}

{% include message.html type="danger" content=message %}

The final step is to point two subdomains `media.example.com` and `play.example.com` to your server, since your DNS will not be configured for this out of the box, you're going to need to set it up yourself, to do this, we suggest making use of CloudFlare, which is a free CDN with support for custom DNS.

**First** create an account at [cloudflare.com](https://www.cloudflare.com/a/signup){:target="_blank"} if you don't already have one, and add your new domain you purchased from namecheap (or wherever).

Once you've setup your domain with CloudFlare and updated your DNS with namecheap (or whoever your domain registrar is), head over to the DNS page for your domain on the [CloudFlare control panel](https://www.cloudflare.com/a/overview){:target="_blank"}. At a **bare minimum** for your private server to function properly you'll need the following A records.

![a records](/assets/images/ubuntu/cloudflare.png)

Where `example.com` is your domain name and `127.0.0.1` is the IP address of your VPS or dedicated server.

{% capture message %}
See those orange clouds on the right? If those are grey you are not benefitting from CloudFlare's **free** CDN, which will speed up your server significantly, make sure to turn on the CDN by clicking the clouds to make them orange!
{% endcapture %}
{% include message.html type="info" content=message %}

This is a breif overview of how to configure CloudFlare, however CloudFlare provides a [much more detailed setup guide here](https://support.cloudflare.com/hc/en-us/categories/200275218-Getting-Started){:target="_blank"}.

### Running Houdini

First, navigate to the Houdini installation directory and check Houdini is actually installed correctly.

```shell
cd ~/houdini
python World.py
```

You should see an output similar to this.

```
INFO:Houdini:Houdini module initialized
INFO:Houdini:5299 items loaded
INFO:Houdini:1384 furniture items loaded
INFO:Houdini:96 igloo items loaded
INFO:Houdini:24 floor items loaded
INFO:Houdini:395 pins loaded
INFO:Houdini:242 stamps loaded
INFO:Houdini:509 cards loaded
INFO:Houdini:6 dance tracks loaded
INFO:Houdini:Handler modules loaded
INFO:Houdini:Running world server
INFO:Houdini:Starting server..
INFO:Houdini:Listening on port 9875
INFO:Houdini:Bot plugin has been loaded
INFO:Houdini:Commands plugin has been loaded!
INFO:Houdini:Rank plugin is ready!
```

If you see anything else, the installation has gone wrong somewhere.

Now press `Ctrl+C` (or `cmd C` on MacOS) to kill the server.

Next, you probably want to make it so Houdini will continue running even after you close your command prompt, to do this, you can use `screen`.

```shell
cd ~/houdini
screen -S login python Login.py
# Now press Ctrl+A D (cmd A D on Mac OS) to detach from session
screen -S world python World.py
# Now press Ctrl+A D (cmd A D on Mac OS) to detach from session
```

Now you may exit your command prompt or terminal. Houdini is installed.

### Test your server

![testing](/assets/images/ubuntu/testing.png)

Yay! It's finally time to test your server! If you've configured your server locally, simply go to `http://play.localhost` in your web browser!
{% capture message %}
If you're doing this inside of a virtual machine, you must visit your server from **within** the virtual machine, that is, open a web browser inside the virtual machine and visit it there, as it has its own loopback address, you cannot go to your server from the host (your PC).
{% endcapture %}
{% include message.html type="danger" content=message %}

If you're hosting publicly, you can visit the server via the domain you purchased (ex. `http://play.example.com`). If you can't reach your server, don't worry, the DNS can take up to 24 hours to change. Afterwards, you may invite your friends to play via your unique domain name!

Still having trouble? No worries, you can get help from real humans on the [Discord](https://discordapp.com/invite/UPnWKfh){:target="_blank"}.

Login with the test penguin account.

_Username_: `Basil`

_Password_: `password`

## Post setup

Congratulations! You've created your first Club Penguin private server! Now that you're done doing that, there's a couple of other things for you to master.

### User registration

At the moment, nobody can register for your private server, [click here](/tutorial/legacy/registration) to learn how to setup a basic registration page.

### Managing the database
Taking control of your database is very useful for managing your player accounts.
#### DataGrip
[DataGrip](https://www.jetbrains.com/datagrip/){:target="_blank"} is a program made by JetBrains which allows you to see a graphical representation of your database, making it super easy to manage. DataGrip is the reccomended way to manage your database since you can connect via an SSH tunnel, which is considered by many to be the most secure way to connect to a remote MySQL database.

[See here](https://www.jetbrains.com/help/datagrip/connecting-to-a-database.html#connect_via_ssh){:target="_blank"} to learn how to connect to your database via SSH.

#### PHPMyAdmin

PHPMyAdmin is considered a fanstastic and easy way to manage your database remotely. It is a web-based application used by many developers around the world. [See here](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-with-nginx-on-ubuntu-16-04){:target="_blank"} to learn how to setup PHPMyAdmin with nginx.

### Managing Houdini
#### Restarting Houdini
Sometimes it is necessary to restart your Houdini server(s) for some reason. This can be done by stopping the screen session and running Houdini again.

```shell
# Stop houdini
screen -r world
# Now press Ctrl+C to kill Houdini world server gracefully
screen -r login
# Now press Ctrl+C to kill Houdini login server gracefully

# Start houdini
cd ~/houdini
screen -S login python Login.py
# Now press Ctrl+A D (cmd A D on Mac OS) to detach from session
screen -S world python World.py
# Now press Ctrl+A D (cmd A D on Mac OS) to detach from session
```
&nbsp;

#### Installing plugins
There is currently no installer or central repository for Houdini plugins (although it has been planned).

If you do have plugins which you wish to install, they can be installed simply by copying them to `~/houdini/Houdini/Plugins/`. Houdini can hot-load new plugins so a restart is not normally required.

Example plugin directory structure:

```
Plugins $ tree
.
├── Bot
│   ├── __init__.py
```
&nbsp;

#### Updating Houdini
Updating Houdini is super easy thanks to Git.

```
cd ~/houdini
git stash
git pull
git stash pop
```

After updating you normally want to [restart Houdini](#restarting-houdini)

### Securing your server
{% capture message %}
You do not need to follow these steps if you are **not** hosting **publicly**, skip this step if you are hosting locally!
{% endcapture %}

{% include message.html type="danger" content=message %}

As a basic security precaution, it is important that you setup a firewall.

```shell
sudo apt-get install ufw
sudo ufw allow 6112,9875,22,80,443/tcp
sudo ufw enable
```

There are [**many** other security precautions](https://www.google.com/search?q=secure+ubuntu+server){:target="_blank"} you can take to secure your server from attackers, and you must take some time now to make sure your server is safe for users to play.

## Miscellaneous

### Reccomended hosting providers
These VPS hosting providers are reccomended based on the community's experience with them. When looking to host a private server you need a provider that won't suspend you without notice, and which provides good value for money, these providers have a good reputation within our community and within the private server community in general, they also have a very wide product portfolio.
- [OVH](https://www.ovh.co.uk/vps/vps-ssd.xml){:target="_blank"}
- [Kimsufi](https://www.kimsufi.com/en/vps-ssd.xml){:target="_blank"}
- [Secure Dragon LLC](https://securedragon.net/#kvm){:target="_blank"}
- [BlazingFast](https://blazingfast.io/vps){:target="_blank"}

### Other useful resources
These resources and tools are not required for this tutorial, but you will find that they will come in handy.
- [Adobe Flash CS6](https://helpx.adobe.com/uk/x-productkb/policy-pricing/cs6-product-downloads.html){:target="_blank"} *(Adobe Flash CC, Adobe Animate, and any version of Adobe Flash newer than CS6 is no use to you for private server development.)*
- [JPEX Free Flash Decompiler](https://www.free-decompiler.com/flash/){:target="_blank"} - Perhaps the most useful tool in the world for dealing with SWF files.
- [FileZilla](https://filezilla-project.org/){:target="_blank"} - Or any FTP client, just like SSH but gives you a GUI to view & manage files on your server.

You should know that this tutorial does not go into exhaustive detail about running and maintaining a private server. It is assumed you are doing this for fun, and as a learning experience, chances are that if you needed this tutorial, you won't be creating the next big Club Penguin private server to replace Club Penguin.
