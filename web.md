---
layout: default
---

## Install a website for your SWG server

Let's say you followed my [guide to build a new VM](./new.html) and now you want a website for your server. Maybe you want users to be able to sign up for an account and only be able to log in if their password is correct. That's what we'll do here.

### Install the software you need

Let's start by installing a web server and a database.

```
sudo apt update
sudo apt install apache2 php libapache2-mod-php php-mysql mariadb-server mariadb-client -y
```

Once that's done, you can verify that your installation worked by visiting your IP address in a web browser.

### Configure your database

Next, we should do a little housekeeping for our database.

```
sudo mysql_secure_installation
```

The first thing you'll be asked is for your root password. You don't have one yet, so just press enter.

Next, it'll ask you if you want to set a root password. Answer yes, then enter a new password and enter it again to confirm. Using the password `swg` is a reasonable suggestion for consistency with the system so far.

You'll now be asked a few more things, but all the defaults are fine so just continue pressing enter until the program finishes.

Now, we'll add a new user for you to login with later.

```
sudo mysql
create user 'swg'@'localhost' identified by 'swg';
grant all on *.* to 'swg'@'localhost' with grant option;
exit;
```

### Install phpmyadmin

Phpmyadmin is a great tool for making and managing your databases.

```
sudo apt install phpmyadmin -y
```

As it's installing, it'll ask you which web server you would like to configure automatically. You want apache2 which is the first option, so press spacebar to select it, then tab to move to the ok button, then enter to select ok.

Next, it'll ask you to configure a database for phpmyadmin with dbconfig-common. This is good, so just press enter. Now, it'll ask you for a MySQL application password. The randomly generated one is fine, so just press tab to move to the ok button, and press enter.

To test your installation, navigate in your web browser to `http://your.ip.address/phpmyadmin` and login with username `swg` and password `swg`.

While you're here, select the Databases tab. Under Create database, enter the Database name `wordpress` and select the Collation `utf8mb4_general_ci`. Click Create.

### Install Wordpress

Wordpress is a pretty good content management system for building a website, and I also have a plugin for it that integrates it with a SWG Server. Install Wordpress like so:

```
cd /var/www/html/
sudo rm *
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzf latest.tar.gz
sudo mv wordpress/* .
sudo rm -rf wordpress latest.tar.gz
sudo chown -R www-data: .
```

Now, navigate to your IP address in your web browser to launch the Wordpress installer.

Select your language and click Continue.

The next page is just informational, so click Let's go.

Change the username and password to `swg`. Leave the other options at their defaults. Click Submit.

If everything seems okay, it'll ask you to click "Run the installation".

Next, you can fill out pretty much whatever you want. Your username should be your SWG account name, if you already have one. Click "Install Wordpress". It'll take just a moment...

When it's done, click Log In and login with the username and password you set on the last page. You'll arrive at your admin dashboard.

Go to Plugins -> Add New. Run a search for "SWG Auth" and my plugin should come up. Click Install Now. Once it installs, click Activate. Now navigate to SWG Auth -> Server Conifg. Here, you'll be given the various lines of configuration that you should add to your SWG Server. The stuff under `[LoginServer]` will already be in servercommon.cfg so you should just edit those lines. The stuff under `[ServerUtility]` is new and can be added directly underneath the other lines you edited.

That's it! Now, users can sign up for SWG on your Wordpress website. SWG will check usernames and passwords with Wordpress to make sure they're valid before letting anyone in. And anyone who is an administrator in Wordpress will be able to access God Mode in game. Keep this plugin up-to-date for new features that are coming soon.
