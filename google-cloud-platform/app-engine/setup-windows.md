---
layout: default
title: "Set Up Windows for App Engine Development"
short-title: "Setup"
description: "How to set up the Windows for App Engine Managed VMs to run Dart programs." 
---
  
# {{ page.title }}

Use the following instructions to set up a Windows machine for
App Engine development.

## Download and install Docker and related tools
{: .no_toc}

* Go to the Docker website and follow the
  <a href="http://docs.docker.com/installation/windows/">
  installation instructions</a>.
  This installs VirtualBox, docker,
  the boot2docker management tool, and a few other
  programs you need.

<aside class="alert alert-warning" markdown="1">
**Note:**
These instructions assume that v1.7.1 of Docker and boot2docker 
are installed on your machine.
However, Docker 1.7.0 is recommended inside the boot2docker VM.
The configuration of boot2docker below handles this.
</aside>

* Put the VirtualBox tool, VBoxManage, in your path.
  The default installation directory is
  `C:\Program Files\Oracle\VirtualBox\`. 

## Configure Docker
{: .no_toc}

<strong>Note</strong>: On Windows,
the docker commands below should be 
run inside the VM as the installation process does not install
a Windows docker command line tool.

  <ol markdown="1">
  <li markdown="1">Run the following commands to configure `boot2docker`.
<pre>
> mkdir "%USERPROFILE%\.boot2docker"
> echo ISOURL = "https://github.com/boot2docker/boot2docker/releases/download/v1.7.0/boot2docker.iso" > "%USERPROFILE%\.boot2docker\profile"
> "%ProgramFiles%\Boot2Docker for Windows\boot2docker" init
</pre>
  </li>
  <li markdown="1">Run the following command to launch boot2docker.
<pre>
> "%ProgramFiles%\Boot2Docker for Windows\boot2docker" up
</pre>

When successful, the output should include lines similar to the following:

<pre>
Docker client does not run on Windows for now. Please use
  "C:\Program Files\Boot2Docker for Windows\boot2docker.exe" ssh
to SSH into the VM instead.
</pre>
  </li>

  <aside class="alert alert-info" markdown="1">
  **Tip:** After running this command, launch VirtualBox.
  You should be able to see boot2docker running.
  </aside>
  </ol>

## Get the Docker images
{: .no_toc}

<ol markdown="1">
<li markdown="1"> `DOCKER_TLS_VERIFY`, `DOCKER_HOST`, and `DOCKER_CERT_PATH`.
   Unfortunately, boot2docker does not support Windows that well here.

First, run the following command:

<pre>
> "%ProgramFiles%\Boot2Docker for Windows\boot2docker.exe" shellinit
</pre>

This prints three lines like this:

<pre>
    export DOCKER_HOST=...
    export DOCKER_CERT_PATH=...
    export DOCKER_TLS_VERIFY=...
</pre>

If you are using the normal Windows shell, take each of these commands
and replace `export` with `set`. For example, for `DOCKER_TLS_VERIFY`:

<pre>
set DOCKER_TLS_VERIFY=1
</pre>

If you are using a cygwin shell, you can use the following script:

<pre>
$ $(boot2docker shellinit)
</pre>

</li>

<li markdown="1">Run the following command to pull a number of Docker
    images required for App Engine Managed VMs.
    The `docker pull` command can take awhile.

<pre>
> "%ProgramFiles%\Boot2Docker for Windows\boot2docker.exe" ssh
$ docker pull google/docker-registry
</pre>
</li>

<li markdown="1"> Check to make sure that you have some images:

<pre>
$ docker images
</pre>

This command lists a number of images.
</li>

<li markdown="1"> Run the following command to print the version of the
    Dart VM. 

<pre>
$ docker run google/dart /usr/bin/dart --version
</pre>

The output will look something like the following:

<pre>
Dart VM version: 1.7.2 (Tue Oct 14 12:12:42 2014) on "linux_x64"
</pre>
</li>

<li markdown="1"> Leave the boot2docker VM:

<pre>
$ exit
</pre>
</li>
</ol>
