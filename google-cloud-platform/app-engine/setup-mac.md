---
layout: default
title: "Set Up Mac for App Engine Development"
short-title: "Setup"
description: "How to set up the Mac for App Engine Managed VMs to run Dart programs."
---

# {{ page.title }}
{: .no_toc}

### Contents
{: .no_toc}

{% include default_toc.html %}

Use the following instructions to set up a Mac for
App Engine development.

## Download and install Docker and related tools

  Run the boot2docker installer from
  <a href="https://github.com/boot2docker/osx-installer/releases" target="_blank">Github</a>.
  This downloads and installs the following tools on your development machine:

* VirtualBox (installed in `Applications` folder)
* `boot2docker` command (installed in `/usr/local/bin`)
* `docker` command (installed in `/usr/local/bin`)

<aside class="alert alert-warning" markdown="1">
**Note:**
These instructions assume that v1.7.1 of Docker and boot2docker 
are installed on your machine.
However, Docker 1.7.0 is recommended inside the boot2docker VM.
The configuration of boot2docker below handles this.
</aside>

## Configure Docker

  <ol markdown="1">
  <li markdown="1">Run the following commands to configure `boot2docker`.
<pre>
$ mkdir ~/.boot2docker
$ echo 'ISOURL = "https://github.com/boot2docker/boot2docker/releases/download/v1.7.0/boot2docker.iso"' > ~/.boot2docker/profile
$ boot2docker init
</pre>
  </li>
  <li>Run the following command to launch boot2docker.
<pre>
$ boot2docker up
</pre>

When successful, the command prints some setup information.
You don't need to type these commands if you use the command in the
following section.
  </li>

  <aside class="alert alert-info" markdown="1">
  **Tip:** After running this command, launch VirtualBox.
  You should be able to see boot2docker running.
  </aside>
  </ol>

## Get the Docker images

<ol markdown="1">
  <li markdown="1">Call the following script to set the required environment
      variables: `DOCKER_TLS_VERIFY`, `DOCKER_HOST`, and `DOCKER_CERT_PATH`.

<pre>
$ $(boot2docker shellinit)
</pre>
  </li>
  <li markdown="1">Run the following command to pull a number of Docker
      images required for App Engine Managed VMs.
      The `docker pull` command can take awhile.
<pre>
$ docker pull google/docker-registry
</pre>
  </li>

  <li>Check to make sure that you have some images:
<pre>
$ docker images
</pre>
  This command lists a number of images.
  </li>

  <li>Run the following command which prints version information about the
      Dart VM. 

<pre>
$ docker run google/dart /usr/bin/dart --version
</pre>
  </li>
</ol>
