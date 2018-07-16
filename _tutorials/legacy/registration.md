---
title: Registration
layout: tutorial
permalink: /tutorial/legacy/registration
menu:
    introduction:
        name: "Introduction"
        requirements: "Requirements"
    registration-setup: 
        name: "Registration Setup"
        installing-php: "Installing PHP"
        configuring-nginx: "Configuring NGINX"
        download-script: "Download script"
        embedding-recaptcha: "Embedding reCAPTCHA"
        configuring-registration-script: "Configuring registration script"
        finish: "Finish"
---

## Requirements

{% capture message %}
**Heads up!** This tutorial assumes you have already setup a private server on Ubuntu using our [setup tutorial](/tutorial/legacy/ubuntu).
{% endcapture %}

{% include message.html type="danger" content=message %}

Registration to your private server can be done by creating a new record in the `penguin` table inside your Houdini MySQL database. You can use [this script](https://gist.github.com/ketnipz/048740d381b454e95afe910fb112ef53){:target="_blank"} to emulate the old flash Club Penguin registration form, but you may also write your own registration form if you are familiar with programming by using this [explanation of Houdini password generation](/password){:target="_blank"}. For the purpose of this tutorial, we'll just setup the emulation script.

![registration](/assets/images/registration/create_penguin.png)

## Installing PHP

First, you must install PHP packages required to execute the script for nginx.

```shell
sudo apt-get install php php-curl php-mysql php-fpm php-mbstring postfix
```

You may be asked to enter your password, type the password and press enter. If you are asked if you wish to continue, type `y` and then enter.

Check your PHP version to ensure PHP is installed correctly and that you have version 7.0 or higher by running `php -v`. Make a note of the major and minor PHP version (that is the number before and after the first point `i.e 7.0`).

## Configuring NGINX

Now you need to configure nginx to execute PHP scripts using the PHP FPM. 

```shell
sudo nano /etc/nginx/nginx.conf
```

This will bring up a text editor, use your arrow keys to scroll down the page until you see something that looks like this:

```conf
...
server {
  listen 80;
  server_name   play.localhost

  root /var/www/play
  index index.html index.htm index.php

  location / {
    try_files $uri $uri/ =404;
  }
  ...
```

Below the snippet above, paste the following.

```conf
location ~* \.php$ {
  fastcgi_pass unix:/run/php/php7.0-fpm.sock;
  include         fastcgi_params;
  fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
  fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
}
```

Replace `php7.0-fpm.sock` to match your PHP version, for example, if you're using PHP 7.2, you would use `php7.2-fpm.sock`. 

Hit `Ctrl+X` (`cmd X` on MacOS) and then `y`, then enter to save. Then, reload nginx.

```shell
sudo service nginx reload
```

Run the following command to download and copy the registration script from GitHub into the correct location on your server.

## Download script

```shell
wget https://gist.githubusercontent.com/ketnipz/048740d381b454e95afe910fb112ef53/raw/
bb77195f47a12b79e67e5bb6fdca25067b1bdf6e/create_account.php -O /var/play/create_account/create_account.php
```

## Embedding reCAPTCHA

This registration script requires that you use a reCAPTCHA to protect you from bots. [Go and get invisible reCAPTCHA keys](https://www.google.com/recaptcha/admin){:target="_blank"} from Google.

![captcha](/assets/images/registration/captcha.png)

Fill out the form, make sure **Invisible reCAPTCHA** is checked. Click "Register" and you should now be able to see a site key & a secret key, make a note of these.

![keys](/assets/images/registration/captcha_keys.png)

Next, you must embed the invisible reCAPTCHA on your play page.

```shell
nano /var/www/play/index.html
```

Paste the following snippet anywhere inside the `<head>` section of the page.

```html
<style type="text/css">
  .grecaptcha-badge {
  display: none;
  }
</style>
<script src='https://www.google.com/recaptcha/api.js'></script>
<script type="text/javascript">
  function onSubmit() {
    document.getElementById("game").finishedCaptcha(grecaptcha.getResponse());
    grecaptcha.reset()
  }
</script>
<form>
  <div id='recaptcha' class="g-recaptcha"
    data-sitekey="YOUR_SITE_KEY_HERE"
    data-callback="onSubmit"
    data-size="invisible"></div>
</form>
```

Substitute `YOUR_SITE_KEY_HERE` in the snippet with the site key you got from Google. Hit `Ctrl+X` (`cmd X` on MacOS) and then `y`, then enter to save.

Finally, configure the registration script to match your preferences.

## Configuring registration script

```shell
nano /var/www/play/create_account/create_account.php
```

At the top of the file you will see an array of configuration options. 

Setting | Description | Example
--- | --- | ---
Hostname | The domain name of the site your users visit to access the game **without** any subdomains. | `clubpenguin.com`
External | The play page url. | `http://play.clubpenguin.com`
SecretKey | The secret reCAPTCHA key you got from google. | `6LfEnEcUAAA`...
Activate | Set this to false if you wish users to verify their email before being able to play. | `false`
Cloudflare | Set this to true if your site is using Cloudflare. | `false`
Approve | If set to false, usernames will appear as user ID until approved manually in database. | `false`
Clean | If set to true, accounts that are not activated will be deleted after a set time period. | `true`
CleanDays | Number of days before inactivated accounts are deleted. | `10`
ForceCase | If set to true, usernames will have the first character capitalised regardless of how the user enters it. | `true`
AllowedChars | Characters which are allowed to be used in usernames. | `A-Za-Z0-9.`
EmailWhitelist | List of email domains which can be used, will be ignored if left blank. | `['gmail.com', 'hotmail.com']`
MaxPerEmail | Number of accounts which can be registered on a single email address. | `5`
Database host | The host address of the MySQL database, normally left as `127.0.0.1`. | `127.0.0.1`
Database user | The username of the MySQL account, normally left as `root`. | `root`
Database pass | The password of the MySQL user account. | `password`
Database name | The name of the database. | `houdini`

Once you have configured your database password, and other relavent information, hit `Ctrl+X` (`cmd X` on MacOS) and then `y`, then enter to save.

## Finish

That's it! You're done, you should now be able to register accounts.

![accountcreation](/assets/images/registration/finish.png) 