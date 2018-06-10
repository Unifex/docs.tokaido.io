---
# Page settings
layout: default
keywords:
comments: false
permalink: faq

# Hero section
title: Frequently Asked Questions

# Micro navigation
micro_nav: true

# Page navigation
page_nav:
    prev:
        content: Containers
        url: '/containers'
    next:
        content: FAQ
        url: '/faq'

---
We hope you enjoy using Tokaido and that it makes your life as a Drupal developer just that little bit easier. If you run into problems, we'd be really grateful if you could share your experience with us so that we can try to help, and make Tokaido just that little bit better.

Before opening a request for help, we respectfully ask that you take a look at the entries in this page, as well as our [FAQ](/faq) to see if your issue has already been discussed. If your issue is there, you'll save a lot of time by looking there first. 

### What versions of Drupal are supported?
Tokaido is tested with the latest supported version of Drupal 8. It should work with older 8.x versions, and with Drupal 7 (where PHP 7.1 is supported), but it's untested with anything other than the latest version of Drupal 8.

### Do I have to put my Drupal site in /docroot? 
Yes. The containers are configured to look for and serve a site from a the `docroot` folder. We might look at implementing a configuration option to change this in future. If you need this feature added, please let us know via [Gitter](https://gitter.im/tokaido-io/Lobby){:target="_blank"}. 

### Do I have to use dynamic port numbers?
No, but it's recommended. We've configured Tokaido to use dynamic port numbers so that multiple projects can run Tokaido at the same time. 

If you want to expose a specific and static port number, you can update the `docker-compose.tok.yml` file. Check out the Docker Compose [ports documentation](https://docs.docker.com/compose/compose-file/compose-file-v2/#ports){:target="_blank"} for more information. 

### How can I bypass the Varnish cache?
You can bypass the Varnish cache (and haproxy) by accessing the Nginx container directly on HTTP. In almost all cases, this shouldn't be necessary. 

You can find out the HTTP port number by running:

- `docker-compose -f docker-compose.tok.yml port nginx 8082`

You can then access that port number in the browser on `http://localhost:{port-number}`

### Can I test my site without SSL? 
Yes, use the same instructions above for bypassing the Varnish cache to access your site directly via Nginx. 

### Is there Drupal Multisite Support?
Yes, [Drupal multisite](https://www.drupal.org/docs/8/multisite/multisite-drupal-8-setup){:target="_blank"} is tested and does work. In order to enable this, you first need to manually create the secondary database. The easiest way to do this is by logging in as the root user to MySQL from inside the Drush container: 

```
ssh {project-name}.tok
mysql -u root -h localhost -p 
```

The password for the root user is `tokaido`. 

You should also manually set up some host file entries on your local system to point your site domains to the Tokaido container. For example, if you had two sites in a multisite setup, one called 'www' and the other 'blog', you could add the following to your `/etc/hosts` file:

```
127.0.0.1 blog.my-site.tok www.my-site.tok
```

<div class="callout callout--info">
    <p>The .tok extension is entirely optional, you can use whatever you like for these hostnames. Just make sure a matching hostname is in your Drupal's sites.php file.</p>
</div>

### What version of Drush is used?
The Drush/SSH container ships with [Drush launcher](https://github.com/drush-ops/drush-launcher){:target="_blank"} which will automatically use the version of Drush you have installed via Composer. 

### Can I stop PHP errors from being displayed in the browser?
Yes. Edit the `docker-compose.tok.yml` file and remove the `environment: PHP_DISPLAY_ERRORS` flag. 

### Why does Tokaido set X-XSS-Protection, HSTS, Referrer-Policy, and X-Frame-Options headers? Can I turn it off?
One of our core design goals with Tokaido was to build a collection of Docker containers that work both in local development, and in production. In fact, we run Tokaido in production successfully. 

We believe that good production environments maximise default security, and this includes sane defaults for HSTS, Referrer-Policy, and other headers. 

These headers are set to their defaults by Varnish, unless they are otherwise set by Drupal. For example, if you want to change the X-Frame-Options header, you can simply specify a different value in Drupal and Varnish will not write a new header. 

The only way to completely bypass these headers is to [bypass the Varnish container](/faq#how-can-i-bypass-the-varnish-cache). 

### Can I run Tokaido in Production?
Absolutely. In fact, the Tokaido Docker images are already in use for production sites, and are built with that goal. 

You can simply re-use the existing images as they are on [Docker Hub](https://hub.docker.com/u/tokaido/dashboard/){:target="_blank"}, or you can fork and modify the source images from [GitHub](https://www.github.com/tokaido-io){:target="_blank"}.

### How well does the filesystem sync process perform compared to other solutions?
In our testing with sites up to 20GB in size containing thousands of files, Unison performs excellently. The only real drawback with Unison, which we think is trivial, is that it has to receive notification that a file has been updated before it can begin the process to replicate it. This means there will always be _some_ kind of delay between writing a file in one system, and having the change replicated to another system. 

After I/O heavy tasks like `composer install`, this delay can become noticeable, but we have never seen this take longer than 1 or 2 seconds, even under the heaviest usage. In daily life for most sites, the replication process in Unison will be unnoticable and the process completed in a fraction of a second.

Most other Drupal local dev solutions (like Beetbox or DrupalVM) rely on a virtual machine to provide the dev environment. This generally results in consistent write operations between the local and virtual system. With these solutions, you'll never see inconsistent content between local and dev environments, but you will see slower read and write performance inside the dev environment. 

Because Tokaido uses Docker, it has the added benefit of using very limited system resources. Each Tokaido environment only needs CPU when your interacting with the Drupal site, and otherwise uses less than 400MB of RAM and 2% of CPU. 

With most small and even medium Drupal sites, you won't notice a performance difference either with Tokaido, or virtual-system based solutions like Beetbox and DrupalVM. 

### How does the SSH config work?
When you run `tok init`, Tokaido installed an SSH profile for your project into `~/.ssh/tok_config`. It looks something like this:

```
Host project-name.tok
    HostName localhost
    Port 32774
    User tok
    IdentityFile ~/.ssh/tok_ssh.key
    ForwardAgent yes
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
```

This makes it easy for you to use SSH with `ssh project-name.tok`. 

### Can I map Drush site aliases from my home directory (~/.drush/) ?
Unfortunately this is a difficult option to implement. Synchronising your `~/.drush` directory would require a separate Unison listener process, which makes each Tokaido environment a little bit more complex and difficult to manage. 

Since the [official example](https://github.com/drush-ops/drush/blob/master/examples/example.site.yml){:target="_blank"} for Drush suggests that the preferred location for site aliases is in the `$ROOT/drush/sites/` folder, we decided to follow this example. 

So in most cases, getting your site aliases to work inside Tokaido is as simple as copying the alias from `~/.drush/` to `$ROOT/drush/sites/`
