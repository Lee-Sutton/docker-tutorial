# Docker tutorial
## Prerequisites 
- You'll need to have docker installed to follow along in this tutorial
- https://www.docker.com/products/docker-desktop

## Step 0 What is docker and what are containers
Docker is an open source technology that allows us to build, deploy, and test our code in containers.

A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. 
A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.


source [https://www.docker.com/resources/what-container]

## Overview
- We're going to build a fully functional environment for developing, running, and deploying a django web server with a postgres database all in docker


## Step 1 - Build the django docker image
- To start we're going to build a docker image to run our django app
- Navigate to the `app` directory
- Run the following command to build the docker image and tag it (with the name of your choice)

```bash
docker build . -t django-tutorial
```

- you can see the docker image has been built by running the following command

```bash
docker image ls
```

- after running the command you should see the following:

```
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
django-tutorial                       latest              6e63c91352c0        10 minutes ago      153MB
```


- this command lists all the docker images that are currently available on your machine.



## Step 2 - Run the docker container
- Now that we've built the image, let's run it and take a look inside

```bash
docker run -it django-tutorial sh
```

- this command tells docker to run a docker container based on the `django-tutorial` image we just created
- the `-i` flag tells docker to run it interactively
- the `-t` flag tells docker to use a "Allocate a pseudo-TTY". In other words let us interact with the container like a terminal
- the `sh` command tells docker which command to run. In this case we're telling it to start a shell

- After running the command we have shell access to the docker container
- I like to think of this as sshing into the docker container
- have a look around. Try running python


## Step 3 - Let's take a closer look at the Dockerfile

