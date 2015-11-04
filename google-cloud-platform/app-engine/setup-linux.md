---
layout: default
title: "Set Up Linux for App Engine Development"
short-title: "Setup"
description: "How to set up the Linux for App Engine Managed VMs to run Dart programs."
---

# {{ page.title }}
Use the following instructions to set up a Linux machine for
App Engine development.

<strong>Note:</strong>
You need a 64-bit Linux system to run Dart on App Engine
Managed VMs.

### Download and install VirtualBox
{: .no_toc}

See
<a href="https://www.virtualbox.org/wiki/Linux_Downloads">Download VirtualBox
  for Linux hosts</a>.

### Get boot2docker and Docker 
{: .no_toc}

<aside class="alert alert-info" markdown="1">
**Do I need boot2docker on Linux?**
There are advantages to using boot2docker on Linux even though
it uses a Virtual Machine (VirtualBox) running Linux.

For example, using boot2docker means that the same setup will be used
on all supported platforms,
and the documentations and answers to questions apply equally no
matter what platform you use. boot2docker also provides more
flexibility and the ability to have several installations with
different versions and a different set of Docker images.
Docker and boot2docker encapsulate everything that you need,
including configuration of the underlying OS,
and allow you to run multiple docker daemons simultaneously.

However, if you want to use a native Docker daemon, that
may be possible. For more information, see
[this question on stackoverflow](http://stackoverflow.com/questions/26842682/dockerdaemonconnectionerror-when-setting-google-cloud-managed-vm-in-ubuntu).
</aside>

Here are the instructions for installing Docker on Linux:

<aside class="alert alert-warning" markdown="1">
**Note:**
These instructions assume that v1.7.1 of Docker and boot2docker 
are installed on your machine.
However, Docker 1.7.0 is recommended inside the boot2docker VM.
The configuration of boot2docker below handles this.
</aside>

<pre>
$ sudo wget https://github.com/boot2docker/boot2docker-cli/releases/download/v1.7.1/boot2docker-v1.7.1-linux-amd64 -O /usr/local/bin/boot2docker
$ sudo chmod 755 /usr/local/bin/boot2docker
$ sudo wget https://get.docker.io/builds/Linux/x86_64/docker-latest -O /usr/local/bin/docker
$ sudo chmod 755 /usr/local/bin/docker
</pre>

### Configure boot2docker
{: .no_toc}

Run the following commands to configure boot2docker:

<pre>
$ mkdir ~/.boot2docker
$ echo 'ISOURL = "https://github.com/boot2docker/boot2docker/releases/download/v1.7.0/boot2docker.iso"' > ~/.boot2docker/profile
$ boot2docker init
$ boot2docker up
</pre>

### Get the Docker images
{: .no_toc}

<ol markdown="1">
  <li markdown="1">Run the following command to set the required environment
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

  <li>Run the following command to print the version of the
      Dart VM. 

<pre>
$ docker run google/dart /usr/bin/dart --version
</pre>
      The output will look something like the following:
<pre>
Dart VM version: 1.7.2 (Tue Oct 14 12:12:42 2014) on "linux_x64"
</pre>
  </li>
</ol>
