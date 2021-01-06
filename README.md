# docker-guide

[Install](#instllation)

## installation

Linux (debian)

[Official-documentation](https://docs.docker.com/engine/install/ubuntu/)

First remove old installation

```console
sudo apt-get remove docker docker-engine docker.io containerd runc
```

New installation (repository setup)

```console
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Install docker engine

```console
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Test docker

```console
sudo docker run hello-world
```

Set permission (optional to use without sudo)

```console
sudo usermod -aG docker $USER
```

Restart PC

#### Images

Images are one of the two core building blocks Docker is all about (the other one is "Containers"). Images are blueprints / templates for containers. They are read-only and contain the application as well as the necessary application environment (operating system, runtimes, tools,...).

Images do not run themselves, instead, they can be executed as containers.
Images are either pre-built (e.g. official Images you find on DockerHub) or you build your own Images by defining a Dockerfile.
Dockerfiles contain instructions which are executed when an image is built ( docker build . ), every instruction then creates a layer in the image. Layers are used to efficiently rebuild and share images.
The CMD instruction is special: It's not executed when the image is built but when a container is created and started based on that image.

#### Containers

Containers are the other key building block Docker is all about.
Containers are running instances of Images. When you create a container (via docker run ), a thin read-write layer is added on top of the Image.
Multiple Containers can therefore be started based on one and the same Image. All
Containers run in isolation, i.e. they don't share any application state or written data. You need to create and start a Container to start the application which is inside of a Container. So it's Containers which are in the end executed - both in development and production.

#### Key Docker Commands

For a full list of all commands, add --help after a command - e.g.

```console
docker --help
docker run --help
```

**_docker build ._** : Build a Dockerfile and create your own Image based on the file

**_-t NAME:TAG_** : Assign a NAME and a TAG to an image

**_docker run IMAGE_NAME_** : Create and start a new container based on image IMAGENAME (or use the image id)

**_--name NAME_** : Assign a NAME to the container. The name can be used for stopping and removing etc.

**_-d_** : Run the container in detached mode - i.e. output printed by the container is not visible, the command prompt / terminal does NOT wait for the container to stop

**_-it_** : Run the container in "interactive" mode - the container / application is then prepared to receive input via the command prompt / terminal. You can stop the container with CTRL + C when using the -it flag

**_--rm_** : Automatically remove the container when it's stopped

**_docker ps_** : List all running containers

**_-a_** : List all containers - including stopped ones

**_docker images_** : List all locally stored images

**_docker rm CONTAINER_** : Remove a container with name CONTAINER (you can also use the container id)

**_docker rmi IMAGE_** : Remove an image by name / id

**_docker container prune_** : Remove all stopped containers

**_docker image prune_** : Remove all dangling images (untagged images)

**_-a_** : Remove all locally stored images

**_docker push IMAGE_** : Push an image to DockerHub (or another registry) - the image name/tag must include the repository name/ url

**_docker pull IMAGE_** : Pull (download) an image from DockerHub (or another registry) - this is done automatically if you just docker run IMAGE and the image wasn't pulled before

## Data & Volumes

Images are read-only - once they're created, they can't change (you have to rebuild them to update them). Containers on the other hand can read and write - they add a thin "read-write layer" on top of the image. That means that they can make changes to the files and folders in the image without actually changing the image.
But even with read-write Containers, two big problems occur in many applications using
Docker:

1. Data written in a Container doesn't persist: If the Container is stopped and removed, all data written in the Container is lost

2. The container Container can't interact with the host filesystem: If you change something in your host project folder, those changes are not reflected in the running container. You need to rebuild the image (which copies the folders) and start a new container Problem 1 can be solved with a Docker feature called "Volumes". Problem 2 can be solved by using "Bind Mounts".

#### Volumes

Volumes are folders (and files) managed on your host machine which are connected to folders /files inside of a container.

There are two types of Volumes:

**_Anonymous Volumes:_** Created via -v /some/path/in/container and removed
automatically when a container is removed because of --rm added on the docker run
command

**_Named Volumes:_** Created via -v some-name:/some/path/in/container and NOT
removed automatically

With Volumes, data can be passed into a container (if the folder on the host machine is not empty) and it can be saved when written by a container (changes made by the container are reflected on your host machine).
Volumes are created and managed by Docker - as a developer, you don't necessarily know
where exactly the folders are stored on your host machine. Because the data stored in there is not meant to be viewed or edited by you - use "Bind Mounts" if you need to do that! Instead, especially Named Volumes can help you with persisting data.
Since data is not just written in the container but also on your host machine, the data survives even if a container is removed (because the Named Volume isn't removed in that case). Hence you can use Named Volumes to persist container data (e.g. log files, uploaded files, database files etc)- Anonymous Volumes can be useful for ensuring that some Container-internal folder is not overwritten by a "Bind Mount" for example.
By default, Anonymous Volumes are removed if the Container was started with the --rm option and was stopped thereafter. They are not removed if a Container was started (and then removed) without that option.
Named Volumes are never removed, you need to do that manually (via docker volume rm
VOL_NAME , see reference below).

#### Bind Mounts

Bind Mounts are very similar to Volumes - the key difference is, that you, the developer, set the path on your host machine that should be connected to some path inside of a Container.
You do that via -v /absolute/path/on/your/host/machine:/some/path/inside/of/container .
The path in front of the : (i.e. the path on your host machine, to the folder that should be shared with the container) has to be an absolute path when using -v on the docker run command. Bind Mounts are very useful for sharing data with a Container which might change whilst the container is running - e.g. your source code that you want to share with the Container running your development environment.
Don't use Bind Mounts if you just want to persist data - Named Volumes should be used for
that (exception: You want to be able to inspect the data written during development).
In general, Bind Mounts are a great tool during development - they're not meant to be used in production (since you're container should run isolated from it's host machine).

#### Key Docker Commands

**_docker run -v /path/in/container IMAGE :_** Create an Anonymous Volume inside a
Container

**_docker run -v some-name:/path/in/container IMAGE :_** Create a Named Volume (named
some-name ) inside a Container

**_docker run -v /path/on/your/host/machine:path/in/container IMAGE :_** Create a Bind
Mount and connect a local path on your host machine to some path in the Container

**_docker volume ls :_** List all currently active / stored Volumes (by all Containers)

**_docker volume create VOL_NAME :_** Create a new (Named) Volume named VOL_NAME . You
typically don't need to do that, since Docker creates them automatically for you if they don't exist when running a container

**_docker volume rm VOL_NAME :_** Remove a Volume by it's name (or ID)

**_docker volume prune :_** Remove all unused Volumes (i.e. not connected to a currently
running or stopped container)

## Docker Compose

Docker Compose is an additional tool, offered by the Docker ecosystem, which helps with orchestration / management of multiple Containers. It can also be used for single Containers to simplify building and launching.

Why? Consider this example:

```console
docker network create shop
docker build -t shop-node .
docker run -v logs:/app/logs --network shop --name shope-web shop-node
docker build -t shop-database
docker run -v data:/data/db --network shop --name shop-db shop-database
```

This is a very simple (made-up) example - yet you got quite a lot of commands to execute and
memorize to bring up all Containers required by this application.
And you have to run (most of) these commands whenever you change something in your code or you need to bring up your Containers again for some other reason.
With Docker Compose, this gets much easier.

You can put your Container configuration into a docker-compose.yaml file and then use just one
command to bring up the entire environment: docker-compose up .

A docker-compose.yaml file looks like this:

```yml
version: '3.8' # version of the Docker Compose spec which is being used

services: # "Services" are in the end the Containers that your app needs
  web:
    build: # Define the path to your Dockerfile for the image of this container
      context: .
      dockerfile: Dockerfile-web
    volumes: # Define any required volumes / bind mounts
      - logs:/app/logs
  db:
    build:
      context: ./db
      dockerfile: Dockerfile-web
    volumes:
      - data:/data/db
```

You can conveniently edit this file at any time and you just have a short, simple command which you can use to bring up your Containers:

```console
docker-compose up
```

You can find the full (possibly intimidating - you'll only need a small set of the available options though) list of configurations here: https://docs.docker.com/compose/compose-file/

Important to keep in mind: When using Docker Compose, you automatically get a Network
for all your Containers - so you don't need to add your own Network unless you need multiple Networks!

#### Installation

These steps should get you there:

```console
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# to verify:
docker-compose --version
```

Also see: https://docs.docker.com/compose/install/

#### Docker Compose Key Commands

There are two key commands:

**_docker-compose up :_** Start all containers / services mentioned in the Docker Compose file
**_-d :_** Start in detached mode
**_--build :_** Force Docker Compose to re-evaluate / rebuild all images (otherwise, it only does that if an image is missing)

**_docker-compose down :_** Stop and remove all containers / services
**_-v :_** Remove all Volumes used for the Containers - otherwise they stay around, even if the Containers are removed.

official command reference: https://docs.docker.com/compose/reference/
