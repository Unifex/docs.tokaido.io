---
# Page settings
layout: default
keywords:
comments: false
permalink: installation

# Hero section
title: Installation
description: Tokaido is available on MacOS or Linux

# Micro navigation
micro_nav: true

# Page navigation
page_nav:
    prev:
        content: Introduction
        url: '/introduction'
    next:
        content: Launching Environments
        url: '/environments'
---

Tokaido can run on both Linux or MacOS if you have Docker installed. There are some additional requirements you'll need to satisfy

# Installing on MacOS
First, make sure you satisfy the system requirements:

- MacOS High Sierra. Older versions may work, but they're untested.
- [Docker for Mac](https://www.docker.com/docker-mac){:target="_blank"}
- [Homebrew](https://brew.sh/) package manager

Installing Tokaido on MacOS is easy, just run:

```
brew install tokaido-io/tok/tok
```

# Installing on Linux
Most Linux distributions should work with Tokaido, but first you'll need to satisfy the system requirements:

- Either a native Docker installation, or Docker Machine (we recommend native CE for better performance)
- [Unison File Synchronizer](https://www.cis.upenn.edu/~bcpierce/unison/){:target="_blank"}
- [fsmonitor](https://pypi.org/project/fsmonitor/){:target="_blank"} (Install with `pip install fsmonitor`)

Once you've satisfied the requirements, you can simply [download Tokaido from https://releases.tokaido.io/tok-latest.tar.gz](releases.tokaido.io/tok-latest.tar.gz). 

Place the downloaded binary in your 

# Building from Source
You'll need Golang for this. 

- Clone [the CLI repository](https://github.com/tokaido-io/tokaido-cli)
- Run `dep ensure`
- Run `go build` 

That's all there is to it! 
