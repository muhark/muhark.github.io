---
layout: post
title: "Making Your Life Easier with Docker (pt 1)"
date: 2021-07-10 12:00:00 +0100
categories: [bash, development, linux, docker]
---

The first hurdle to becoming a computational social scientist is downloading and installing all the tools you need to begin writing code for your analyses. Unlike the act of coding itself, the task of managing your tools and keeping your libraries up-to-date is not something that becomes easier over time. In fact, the opposite is often true. How often have you wanted to send a colleague some code, or run replication code, only to find that your version of R is incompatible with a necessary library?

Managing your _environment_ –the stack of software that your programs run on–is a time-consuming, necessary and largely unrewarding task. Most of the time, it simply is a periodic obstacle to the higher priority task of getting research done. But it's not efficient for all computational researchers to become competent system administrators.

Eventually, all that environment management you've been putting off becomes an obstacle to two important tasks:

- Writing replication material.
- Working in remote servers (cloud or HPC).

In this (and subsequent) posts, I will show you how to make your programming environment "portable" with a tool called Docker. If you already work with Jupyter or RStudio, I think that Docker can offer you a solution that will a) constitute a minimal change to the way you work, b) help you keep your Python/R version up-to-date and c) make it much easier to share code and collaborate.

As usual, I'll try to keep it light on the technical stuff for now, and just focus on how you can get something up and running. Where I go into detail, it will be to help you figure out how to troubleshoot when things go wrong.

# Portable Data Science Environment

This post will:

- Walk you through how I created a portable template for my data analysis tools.
- Focus on the parts of Docker relevant to a social science research use case.
- Provide you with all of the relevant commands to get your own portable environment up and running, and customise it to suit your needs.

Specifically, I'll show you how to have an installation of RStudio or Python+Jupyter that is always up-to-date, without the pain of reconfiguring it each time. I'll try and address other IDEs in a future post.

## Skipping to the End

Use the following command to run Jupyter Lab with Python, R and Julia pre-installed running in your current directory:

~~~{bash}
sudo docker run \
	--rm -p 8888:8888 \
	-v "$(pwd)":/home/jovyan/work:z \
	-e JUPYTER_ENABLE_LAB=yes \
	jupyter/datascience-notebook \
	start-notebook.sh --NotebookApp.token='<password>'
~~~

Then open a browser to `localhost:8888` and log in with the token you set as `<password>`.

To kill the process you can hit Ctrl+C twice.

To customise this setup with your preferred libraries, read the section of this post on _Dockerfiles_.

# Docker

Docker is a _containerization_ software written in the Go language. Although it is often explained with an analogy to Virtual Machines (VMs), which you might be familiar with, I found it more helpful to think of Docker as a way to package your "stack".

In order to run code, we need a "stack" of tools and libraries, from the operating system and the instruction set to the processor to the R and Python binaries and libraries for running our code. For Linux, there are many distributions with different libraries, but the same underlying kernel. Thus in theory, the reason you can run a program on one Linux system but not another is the presence or absence of the relevant libraries and configuration of the environment.

_Note for Windows users:_ This in fact extends to cross-OS cases; you can run Linux code on Windows thanks to the Hyper-V technology that runs a minimal Linux kernel on Windows systems.

As mentioned, installing these libraries and configuring the environment is a pain. So what if we could just "box up" a pre-configured environment and deploy it on a different machine? This is in essence what Docker allows you to do in a lightweight and convenient way. (_Big oversimplification, but will give more details below_.)


## Installing Docker

I installed Docker using the relevant software manager for my distro (Fedora, `dnf`, but nobody asked). I won't go over installation since a) I don't have access to a Mac or Windows system, and b) I assume that my readers are experts at following instructions on websites. Here's the website:

[`https://docs.docker.com/get-docker/`](https://docs.docker.com/get-docker/)

Do let me know what kinds of problems you run into; it'll help me improve this guide.

## Docker Terminology

There are many parts to Docker. Keeping in mind that our goal is to make the standard data analysis toolkit portable, I'll focus on the three fundamental components of Docker that are relevant to this post. These are:

- _Image_
- _Container_
- _Volume_

A **Docker Image** is a pre-packaged software stack, and can be thought of as a template. These can be downloaded from and shared via Docker Hub.

A **Docker Container** is an instance of an image. These are what actually run on your machine. As we'll see, Docker containers are like an isolated environment running inside your computer. Once a container is killed, it is completely erased along with any changes you made to it.

A **Docker Volume** is a persistent storage volume that can be attached to a docker container. This is one of two ways to use 

## Tutorial 1: Downloading and Running an Image

For our first example, I will run you through downloading and running an Ubuntu Docker image.

As an optional first step, to see what images you already have installed, type:

~~~{bash}
docker image list
~~~

**!!!**
_If you run into an issue stating that you do not have permissions to run this command, you should preface all commands in this tutorial with `sudo`. If you do not know what `sudo` does, then you should not use this; you could do serious damage to your system otherwise._

The output will depend on whether you've downloaded a Docker image before. Regardless, the next command will be the same.

The following command will:

- download the latest version of the ubuntu docker image available (if you don't already have it)
- start a container running this image
- provide terminal in this container

~~~{bash}
docker run -it --rm ubuntu:latest
~~~

Breaking down this command:

- `docker run`: start a container
- `-it`: combining two options, `-i` and `-t`:
	- `-i`: start an interactive session
	- `-t`: provide a terminal in the interactive session
- `--rm`: remove the container once the command is finished
- `ubuntu:latest`: the image name. The `:latest` is optional, as by default the latest image is used.

You should now have a prompt that looks something like the following. Note that the string after `root@` will be different:

~~~{bash}
root@390f03a3c1a2:/# 
~~~

To check the distro information, you can `cat /etc/*release`. This should return Ubuntu (among other details).

As I mentioned, changes made inside the container will not persist when the container is killed. To see this for yourself, run the following one-liner to create a file and write "Hello World!" to it.

~~~
echo "Hello World!" > README.txt
~~~

Now check with `pwd`, `ls` and `cat README.txt` to confirm that the file was created at `/README.txt`.

Finally, exit the container with `exit` or Ctrl+D.

Try running the original command again to create a new container:

~~~{bash}
docker run -it --rm ubuntu:latest
~~~

Note that the string after `root@` should be different this time. Now try `ls`; you'll see that `README.txt` is gone!

Why is this the default behaviour? In short, the image is a template, and the container is always created based on the image that it is given. This stateless behaviour is desirable when we want to deploy identical copies of an environment in many different locations.

## Tutorial 2: Non-Interactive Sessions

Usually, we run containers in the background and interact with the application that they're hosting. Let's run the intro to Docker image provided by the official Docker documentation:

~~~{bash}
docker run -dp 8000:80 docker/getting-started
~~~

- `-d` means to "detach" the container and let it run in the background.
- `-p A:B` links port A on the host machine to port B on the container.
- `docker/getting-started` is the "getting started" container provided by Docker themselves.

To see currently running docker containers, we can use the following command:

~~~{bash}
docker container ls
~~~

You should see something like the following:

~~~{bash}
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                NAMES
8afc907d8a70        docker/getting-started   "/docker-entrypoint.…"   5 seconds ago       Up 4 seconds        0.0.0.0:80->80/tcp   wizardly_chaum
~~~

This output describes all of the currently running docker containers:

- `CONTAINER ID`: randomly generated unique ID to refer to the container.
- `IMAGE`: image the container was generated from.
- `COMMAND`: command running in the docker container.
- `CREATED`: time since container was created.
- `STATUS`: 'Up' indicates that the container is active. Note that by default, `docker container ls` only shows active containers. To see inactive containers, add ` -a` after `ls`.
- `NAMES`: if not given, a randomly generated name to refer to the container.

_Note_: Many guides use the older `docker ps` command, short for "process status". Feel free to use either; the `ps` may seem more natural to those familiar with shell tools, while `docker ls`/`list` is consistent with the typology introduced in more recent versions of Docker.

A container has its own ports (if you don't know what these are, you can think of them as communication channels). Remember that when we instantiated the container we linked port 8000 of the host machine (your computer) to port 80 of the container with `-p 8000:80`.

Open up [`localhost:8000`](https://localhost:8000) in your browser and you should see a Docker tutorial! (I found that tutorial helpful, but it's focused on a different use case than the one in this tutorial. That tutorial looks more at containerizing _applications_, as opposed to _environments_.)

To shut down and delete that container, run the following command:

~~~{bash}
docker rm -f <container_name>
~~~

In my case, I wrote `wizardly_chaum` in place of `<container_name>`. What you write will depend on the output of `docker container ls`.

Also note that `rm` means "remove", and `-f` means "force". By default, you cannot remove actively running containers. If you pass `-f` then you can.

# Tutorial 3: Jupyter in Docker

[**Jupyter**](https://jupyter.org/) is a project providing a popular set of IDEs. I sometimes use JupyterLab for development, especially when creating data visualisations. Even when I'm coding in VIM, I use a Jupyter kernel/console combo to have an interactive session to send chunks of code to.

The associated IDEs, Jupyter Notebooks and JupyterLab, are used through web browser on a specified port–by default port 8888. This makes it an easy candidate for using in a Docker container, as it requires relatively little change to the way you may already use Jupyter.

Jupyter provides a variety of pre-configured Docker images. For a guide on the available images, see [here](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html). We're going to use the `jupyter/datascience-notebook`, which comes with common Python, R and Julia data science libraries pre-installed.

Let's start building our one-liner to spin up this container:

~~~{bash}
docker run \ 
	--rm -p 8888:8888 \
	jupyter/datascience-notebook
~~~

Breaking this down:

- _Note_: the backslash allows us to put a command over multiple lines for readability. On Windows powershell you should replace this with backticks.
- `--rm`: remove the container when finished.
- `-p 8888:8888`: linking port 8888 of the host system to port 8888 of the container.
- `jupyter/datascience-notebook`: the name of the image

When you run this, Docker should start pulling the image from Docker Hub. It might take a bit of time since it's a sizable container. (Also be careful not to run out of space on your computer while doing this–a few GB should be adequate).

The images provided by Jupyter have their own customization options, which I'll share now. I prefer using JupyterLab to Jupyter Notebook, and I prefer to set my own password, so I add the following parameters:

~~~{bash}
docker run \
	--rm -p 8888:8888 \
	-e JUPYTER_ENABLE_LAB=yes \
	jupyter/datascience-notebook \
	start-notebook.sh --NotebookApp.token='<password>'
~~~

- `-e JUPYTER_ENABLE_LAB=yes`: a custom parameter to use Lab instead of Notebook.
- `start-notebook.sh --NotebookApp.token='<password>'`: see the documentation for a full list of customisation options. This one lets you set the password for the Notebook/Lab server.

We're more or less good to go! However, recall the previous examples; Docker containers don't have a persistent file system, nor do they have access to the host file system by default. This will become a problem when we try to interact with files on the host system. Likewise, we may want to install additional libraries beyond those that come pre-packaged, and we don't want to have to re-install these each time we spin up a container.

I'll cover how to deal with each of these problems in turn.

## Mounts

Docker provides two options for persistent file storage: _mounts_ and _volumes_. _Mounts_ are a "bridge" between the Docker and the host file system, whereas _volumes_ are a special containerized file system that can be attached/detached to Docker containers. Both are very useful, but we'll just be looking at mounts since these are closer to the standard social science research use case.

In order to mount a file system, we can use the `-v` parameter. Here's how it looks:

~~~{bash}
docker run \
	--rm -p 8888:8888 \
	-v "$(pwd)":/home/jovyan/work:z \
	-e JUPYTER_ENABLE_LAB=yes \
	jupyter/datascience-notebook \
	start-notebook.sh --NotebookApp.token='<password>'
~~~

Breaking down the third line:

- `-v`: Docker parameter for mounting.
- `"$(pwd)"`: shell command that resolves to the current working directory
- `/home/jovyan/work`: the Jupyter image has a non-root user called `jovyan`. The default working directory for this user is the one just listed.
- `:z`: extends the permissions of the `jovyan` user onto the directory that it has been mounted on. Without this, you'll be unable to make any changes on the host file system from a user within the container.

I recommend that you `cd` (change directory) into the directory where you'll be doing your analysis and then run this command. You can always substitute `$(pwd)` for any valid path on the host file system.

## Installing Libraries at Runtime

In order to install additional libraries at runtime, we could use Jupyter's functionality for executing shell code by prefixing commands with `!`, but I prefer to do everything directly in a terminal. Fortunately getting a terminal on a running container is relatively straightforward.

After obtaining the name of the container with `docker container ls`, you can use the following command to open a terminal in the container:

~~~{bash}
docker exec -it <container_name> /bin/bash
~~~

- `exec`: Docker command for executing commands in running containers
- `-i`: stands for "interactive"
- `-t`: stands for "tty" (I think). Essentially a terminal.
- `<container_name>`: use `docker container ls` to get this.
- `/bin/bash`: the command to execute. In this case, executing `/bin/bash` will spawn an interactive shell.

From here, we can execute code as we like on the container. Say we want to install the `nltk` library:

~~~{bash}
pip install nltk
~~~

## Persisting Installed Libraries: Dockerfiles

Recently I've been working a lot with the PyTorch library, but the image I found on Docker Hub had an outdated version of the library, so I created my own Docker-PyTorch-Jupyter image. As you'll see, this is a convenient way to make persistent changes to the installed libraries on a Docker image.

A Dockerfile is a text file with a special set of commands that provides instructions to the `docker build` command. In a directory of your choice (I put mine in the top-level directory of the project where I'm using this image), create a file called `Dockerfile`. Here's what I wrote in mine:

~~~{bash}
FROM jupyter/scipy-notebook:latest
MAINTAINER Musashi Harukawa

RUN pip3 install torch==1.9.0+cpu \
				 torchvision==0.10.0+cpu \
				 torchaudio==0.9.0 \
	-f https://download.pytorch.org/whl/torch_stable.html
RUN pip install nltk
RUN pip install jupyterlab_vim
~~~

Running through the commands:

`FROM` provides a base template for the image. The genius part about this is that it makes building new images very easy, as you can simply build them on top of something that already works.

`RUN` executes a command at build, meaning that each container spawned from this image will have the changes produced by this command. This means that it does _not_ have to run each time that we spin up a container from this image.

Then in that directory, run the following command to build your image:

~~~{bash}
docker build -t pytorch-cpuonly .
~~~

- `build`: the command to create images from Dockerfiles.
- `-t`: provides the "tag", or name for the image.
- `pytorch-cpuonly`: the name I gave to my Docker image. Feel free to change it! Note that you can also suffix this with `:latest` or something else to provide versioning.
- `.`: execute the build in the current directory (where the Dockerfile is).

# Up Next

I only started learning Docker two weeks ago, and it's already a core part of my development setup. In particular, it's made working on multiple machines much smoother (I use my old laptop to dry-run intensive code).

In my next post (hopefully sooner than in a month), I'll give a short demo on how to run RStudio Server in a Docker container. In the meantime, feel free to contact me on Twitter (or elsewhere) to tell me things you want to learn/see!


