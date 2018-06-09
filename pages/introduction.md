---
# Page settings
layout: default
keywords:
comments: false
permalink: introduction

# Hero section
title: Dockerised Drupal 8 Development Environments
description: Welcome to Tokaido

# Search box
author:
    title: Search Box Here
    title_url: '#'
    external_url: true
    description: Will use [Algolia](https://community.algolia.com/docsearch/)

# Micro navigation
micro_nav: true

# Page navigation
page_nav:
    next:
        content: Installation
        url: '/installation'

---

Tokaido (*tok-eye-doh*) is a command-line utility and a collection of open source Docker containers that enable you to quickly and easily launch new local development environments for Drupal 8 on MacOS and Linux.

The Tokaido CLI - `tok` - gives you access to launch, control, and destroy local dev environments with Docker. You can run multiple Tokaido environments simultaneously, one for each Drupal site you're working on.

New Tokaido environments are launched easily with `tok init`.

## What's included

Each Tokaido environment you launch comes complete with:

* A HTTPS-enabled [LEMP Stack](https://lemp.io/){:target="_blank"} (Linux, (E)Nginx, MySQL, and PHP)
* Rapid, bi-directional file-sync between the Tokaido environment and local host
* SSH access to a 'Drush container' for local and remote site management
* Production-friendly defaults that enable things like best-practice XSS and SAME-ORIGIN headers
* A local Varnish cache to test production readiness of your caching config
* A local memcache instance 
* Easy to access logs for Nginx, Varnish, and PHP

## Tokaido Commands

Once your local stack is running, Tokaido provides access to a variety of convenience functions to make it easier to work with and manage your local dev environment, including:

* `tok init` - Start a new Tokaido environment. You want to run this first from your Drupal repository root 
* `tok drupal-setup` Update settings.php in the default site so that it knows how to access the Tokaido-hosted database
* `tok sync` - Perform a one-time sync of between local host and the Tokaido dev environment
* `tok watch` - Maintain an open process to watch for all content changes and actively sync whenever necessary
* `tok open` - Launch your browser pointing to the local Tokaido environment
* `tok syscheck` - Test that your local system has all the necessary bits and pieces to run Tokaido successfully. 
* `tok stop` - Shut down (without deleting) the Tokaido environment, to help save on system resources
* `tok up` - Start a stopped environment (useful after you rebooted)
* `tok destroy` Viciously murder your Tokaido environment :'(

You can run `tok help` at any time to see more commands that Tokaido offers. 

## Drupal Requirements

One of the most beautiful but challenging things about Drupal is that there are so many different ways you can use it. Tokaido is designed to support the most common use case:

- Drupal 8.5.3 or newer is supported (Drupal 7 might work, but it's unsupported)
- Your Drupal site should be installed and managed with Composer (although this isn't essential)
- Your Drupal site must reside in the `docroot` directory

If you can't satisfy these requirements, but would still like to use Tokaido, please [get in touch](). We want to hear from you so that we know that there is demand for other use cases, and we hope we can ask you to help us test support for these new configurations.

<div class="callout callout--success">
    <p><strong>Time to Install?</strong></p>
    <p>Installing the Tokaido CLI is easy. <a href="/installation">Click here</a> to check out the installation instructions</p>
</div>



## How File Sync Works
Tokaido performs a high-speed bi-directional copy between your local system and the Tokaido environment. 

You can perform a one-time sync of your local and Tokaido environments with the `tok sync` command. But you'll probably want to run `tok watch` which will keep running and perform synchronisation in real time. 

Tokaido uses the [Unison File Synchronizer](https://www.cis.upenn.edu/~bcpierce/unison/){:target="_blank"} to keep your local system and the Tokaido environment in perfect sync. Doing things this way enables us to ignore user ownership problems that plague local Docker development environments. You can think of Unison as bi-directional rsync. 

In our testing with a 20GB Drupal site containing thousands of small files, Unison was extremely fast. In most cases, Unison will synchronise your local and Tokaido environments in a fraction of a second. 