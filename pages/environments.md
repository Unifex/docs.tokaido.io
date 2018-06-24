---
# Page settings
layout: default
keywords:
comments: false
permalink: environments

# Hero section
title: Environments
description: Launching and Managing Drupal Environments with Tokaido

# Micro navigation
micro_nav: true

# Page navigation
page_nav:
    prev:
        content: Installation
        url: '/installation'
    next:
        content: Containers
        url: '/containers'

---

<div class="callout callout--warning">
    <p><strong>Drupal 8 Requirements</strong></p>
    <p>If you skipped the bit about <a href="">Drupal 8 Setup Requirements</a> we'd like to point out that Tokaido probably won't work unless you have Drupal configured as follows:</p>
    <p><ul style="font-size: .9375em; line-height: 1.8em;">
        <li>Your site should run Drupal 8.5.3 or newer</li>
        <li>You should be using <a href="https://getcomposer.org" target="_blank">PHP Composer</a></li>
        <li>Your site must reside in the "docroot" directory. This one is essential, sorry.</li>
    </ul></p>
</div>

Each Tokaido local development environment is a collection of Docker containers working together to run your Drupal site.

You can use the Tokaido command line environment (CLI) to easily start and manage these environments. In this guide, we're going to 
show you how to launch and maintain each environment. 

## Creating Environments
There's no real limit to the number of environments you can run at any one time, but you can only create one environment per project. 

Our example commands here will be using the [tokaido-test-project](https://github.com/tokaido-io/tokaido-test-project){:target="_blank"} repository, which you can clone and use for testing if you like. 

To get started:

- Open a terminal window and navigate to your project's repository directory (the "repo root")
- Run `tok up` 

```
$ tok up
🚅  Tokaido is initializing your project!
🏯  Generating a new docker-compose.tok.yml file

WELCOME TO TOKAIDO
==================

Your Drupal development environment is now up and running

⌚  Run "tok watch" to keep files in your local system and the Tokaido environment synchronised
💻  Run "ssh odppnsw.tok" to access the Drush container
🌎  Run "tok open" to open the environment in your browser

Check out https://docs.tokaido.io/environments for tips on managing your Tokaido environment

```

That's all you need to do! As the output suggests, you can access the SSH container and Drush by running `ssh your-project-name.tok`. 

You can also run `tok open` to have the Tokaido site in your browser. But if you've only just run `tok up`, you'll probably just get an error message. Let's move on... 

## Checking the Status of your Environment
You can run `docker-compose -f docker-compose.tok.yml ps` from the repo root at any time to see the status of your Docker containers. 

```
$ docker-compose -f docker-compose.tok.yml ps
       Name                     Command               State                        Ports
--------------------------------------------------------------------------------------------------------------
odppnsw_drush_1      /usr/local/bin/entrypoint.sh     Up      0.0.0.0:32813->22/tcp
odppnsw_fpm_1        /usr/local/bin/entrypoint.sh     Up      0.0.0.0:32815->9000/tcp
odppnsw_haproxy_1    /usr/local/bin/entrypoint.sh     Up      0.0.0.0:32818->8080/tcp, 0.0.0.0:32817->8443/tcp
odppnsw_memcache_1   docker-entrypoint.sh memcached   Up      11211/tcp
odppnsw_mysql_1      docker-entrypoint.sh mysqld      Up      0.0.0.0:32814->3306/tcp
odppnsw_nginx_1      /usr/local/bin/entrypoint.sh     Up      80/tcp, 0.0.0.0:32816->8082/tcp
odppnsw_solr_1       solr-precreate drupal /opt ...   Up      0.0.0.0:32812->8983/tcp
odppnsw_syslog_1     /usr/sbin/syslog-ng -F --p ...   Up      5514/tcp, 5601/tcp
odppnsw_unison_1     /sbin/tini -- /entrypoint.sh     Up      0.0.0.0:32811->5000/tcp
odppnsw_varnish_1    /usr/local/bin/entrypoint.sh     Up      8081/tcp

```

Using this status output, you can see that all the containers in our example are "Up". That means that Tokaido should be working and ready to go. 

The "Ports" output can show you the local port number (immediately following "0.0.0.0:") for some of the exposed services. For example, you can bypass Varnish and Haproxy by connecting directly to the Nginx port on port 32770.

<div class="callout callout--warning">
    <p>Port numbers change every time you start a Tokaido environment using `tok up`.</p> 
    <p>This is why commands like `tok open` make life a bit easier, but you can always look up the current port number with `tok ports` as well.</p>
</div>

## Keeping in Sync
In a new terminal window, run `tok watch` to synchronise files between your local disk and the Tokaido environment. 

If you don't want to keep this running all the time, you can run `tok sync` to perform a one-time sync on demand. 

## Configuring Drupal to use the Tokaido Database
Your site needs to be told how to access the MySQL database that Tokaido provides. 

When you run `tok up` on a new, Tokaido will ask you if you want to automatically configure this database connection. If you agree, Tokaido will create the file `docroot/sites/default/settings.tok.php` and link to it from `docroot/sites/default/settings.php`. It will also add some Tokaido-specific files to your `.gitignore`

You can optionally manually configure your database connection, which can be especially helpful for Drupal multisite configurations.

To do this, add the following line to the bottom of your `sites/default/settings.php` file:

<div class="callout callout--danger">
    <p>Make sure you have `tok watch` running so that your changes are copied into the Tokaido environment</p>
</div>

```php
if (file_exists($app_root . '/' . $site_path . '/settings.tok.php')) {
  include $app_root . '/' . $site_path . '/settings.tok.php';
}
```

Now you can create the `settings.tok.php` file with the following:

```php
<?php
    $databases['default']['default'] = [
    'host' => 'mysql',
    'database' => 'tokaido',
    'username' => 'tokaido',
    'password' => 'tokaido',
    'port' => 3306,
    'driver' => 'mysql',
    'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
    'prefix' => '',
    ];

    $settings['file_private_path'] = $app_root . '/../files-private/';
?>
```

<div class="callout callout--warning">
    <p>You don't want the settings.tok.php file being used in your production environments. Be sure to add this file to your .gitignore</p>
</div>

## Accessing the Environment Using SSH 
It's time to access the Tokaido environment directly, which you can do via SSH.

- Run `ssh project-name.tok`. Substitute 'project-name' as necessary; By default Tokaido uses the directory name of the repo root.

This logs you into a container that shares a copy of your site with the other containers like Nginx. 

```
$ ssh tokaido-test-project.ssh

      PHP Version      : 7.1.17
      Composer Version : 1.6.5
      Drush Version    : 9.2.3
      Database Status  : Failed

The Tokaido local development system is ephemeral. Each time you restart the
local Tokaido instance, your database is reset. Content not saved in the
/tokaido/site folder will also be lost forever.

tok@tokaido:/tokaido/site/docroot$
```
When you first log in, you'll be in the `/tokaido/site/docroot` folder. If you haven't run `composer install` on your Drupal site yet, now is a good time (you'll need to move up one directory).

After Composer, you can run `drush site-install` to install Drupal:

```
🚅 LOCAL project-name 03:53:45 /tokaido/site/docroot $ drush site-install -y

 // You are about to DROP all tables in your 'tokaido' database. Do you want to continue?: yes.

 [notice] Starting Drupal installation. This takes a while. Consider using the --notify global option.
 [success] Installation complete.  User name: admin  User password: wowzer
```

Outstanding! Go ahead and run `drush status` to see how everything looks. 

You can type `exit` at any time to leave the SSH environment and go back to you local system. 

## Accessing Your Site in the Browser
To access your site, run the following command from your host (make sure you exit the SSH container):

- `tok open`

This opens the site via the HAProxy container using HTTPS. 

Since Tokaido uses dynamic port numbers, we added this command to make it easy to open your site in your default browser. On Mac, this runs the `open` command, or on Linux it runs `xdg-open`.

If you can't use `tok open` for some reason, or you want to use a different browser, you can look up the port number with the `tok ports` command.

## Starting and Stopping the Environment
After you log out, reboot, or manually stop the environment, you'll need to start it again. 

- Run `tok up` to start your containers again

If you're short on system resources and want to stop Tokaido, you can!

- Run `tok stop` to pause your containers

While your containers are stopped, they retain their data. This is most important for your database. 

## Destroying and Restarting the Environment
Feeling destructive? Having a bad day? 

- Run `tok destroy` to completely delete and erase your containers. 

<div class="callout callout--warning">
    <p>tok destroy will delete your database!</p> 
</div>

When you want to restart the environment, just run `tok up` again. 

## Modifying the PHP Environment
The Tokaido PHP FPM container ships with some sane defaults that are generally reasonable for both local development and production environments. 

The container is open source, and you can find it [here](https://github.com/tokaido-io/fpm){:target="_blank"}. 

The README for the container lists both the default values, and variable names that you can apply to override them. 

If you want to override any of these settings, you need to manually modify your `docker-compose.tok.yml` file that Tokaido placed in your repo root. These new values are applied as environment variables to the FPM container. 

In the below example, we override the memory_limit and post_max_size variables in PHP. 

<div class="callout callout--info">
    <p>Note that Tokaido overrides the "display_errors" value by default</p>
</div>

```yaml
fpm:
    user: "1001"
    image: tokaido/fpm:0.0.2
    env_file: .env
    working_dir: /tokaido/site/
    volumes:
      - ./.env:/tokaido/config/.env
    volumes_from:
      - unison
      - syslog
    depends_on:
      - syslog
    environment:      
      PHP_DISPLAY_ERRORS: "yes"
      PHP_MEMORY_LIMIT: 128M
      PHP_POST_MAX_SIZE: 10M
```

You'll need to restart the FPM container after you modify this file. You can do that by running `docker-compose -f docker-compose.tok.yml kill fpm && docker-compose -f docker-compose.tok.yml up -d fpm`

<div class="callout callout--warning">
    <p>Values like 1, 0, 'yes', and 'no' need to be quoted when declared in YAML, to avoid them being converted to True or False and breaking the php.ini file.</p>
</div>


## Working with the Varnish cache
Healthy production Drupal environments run Varnish, so Tokaido does too. The Varnish container, like all the other Tokaido containers, is open source. You can view it [here](https://github.com/tokaido-io/varnish){:target="_blank"}. 

Caching in local development can be a bit scary, but the Tokaido Varnish container has three very simple caching rules in the following order:

- If the user is logged in, don't cache
- If Drupal provides a cache header, honour it
- If there's no cache header, then don't cache

So unless Drupal sends cache headers, then nothing will be cached. 

You can see if your content is being cached by looking at the response headers in your browser, and seeing if `X-Varnish-Cache` is set to `MISS` or `HIT`. If `MISS`, then Varnish is delivering you delicious fresh content.

You can manage cache settings for your Drupal site in the `admin/config/development/performance` page. Note that if you change these settings they won't immediately apply and Varnish might still retain cached content. 

We're working on adding support to clear cache dynamically by using Drush, or by running a tok purge command. For now, however, you can only clear cache by restarting the Varnish container with the following manual command:

`docker-compose -f docker-compose.tok.yml kill varnish && docker-compose -f docker-compose.tok.yml up -d varnish`

Varnish also adds some production-ready headers like `X-FRAME-OPTIONS` and `X-XSS-PROTECTION`. Generally these shouldn't even be noticeable, but we set them because we want Tokaido environments to be as production-like as possible, and good production environments set these headers. 

If you want to bypass HAProxy and Varnish, you can access the Nginx container directly on HTTP (not HTTPS). Look up that port number with this command:

- `tok ports nginx`

## Using Drush Aliases
Drush and Drush aliases work perfectly inside Tokaido environments. The only problem you might encounter is that Tokaido doesn't synchronise any location outside of your repo root. This means that aliases stored in your home directory (ie. `~/.drush`) aren't known inside Tokaido. 

The easiest way to address this is to place your alias inside the `drush/sites` folder inside the repo root. 

## Accessing the Database
Tokaido inclues a MySQL container. This isn't anything special we've created, but rather the [official MySQL Docker image](https://hub.docker.com/_/mysql/){:target="_blank"}. 

The MySQL credentials are as follows:

- Username: tokaido
- Password: tokaido
- Hostname: mysql
- Default Database Name: 

You can access the database from inside Tokaido by using either `drush ssh` or `mysql -u tokaido mysql -p `. 

If you want to access the MySQL database from your local system, you need to identify the MySQL Port:

- `tok ports mysql`

Finally, you can log in as root by using the same password (`tokaido`). For example: 

- `mysql -u root -h 127.0.0.1 -P 32779 -p`

<div class="callout callout--info">   
    <p>If you're connecting via the command line, note that you have to use the IP address 127.0.0.1 not the hostname 'localhost'.</p>
</div>