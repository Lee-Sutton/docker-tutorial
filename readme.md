# Docker tutorial

## Prerequisites

-   You'll need to have docker installed to follow along in this tutorial
-   https://www.docker.com/products/docker-desktop

## Step 0 What is docker and what are containers

Docker is an open source technology that allows us to build, deploy, and test our code in containers.

A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.
A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

source [https://www.docker.com/resources/what-container]

## Overview

-   We're going to build a fully functional environment for developing, running, and deploying a django web server with a postgres database all in docker

## Step 1 - Build the django docker image

-   To start we're going to build a docker image to run our django app
-   Navigate to the `app` directory
-   Run the following command to build the docker image and tag it (with the name of your choice)

```bash
docker build . -t django-tutorial
```

-   you can see the docker image has been built by running the following command

```bash
docker image ls
```

-   after running the command you should see the following:

```
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
django-tutorial                       latest              6e63c91352c0        10 minutes ago      153MB
```

-   this command lists all the docker images that are currently available on your machine.

## Step 2 - Run the docker container

-   Now that we've built the image, let's run it and take a look inside

```bash
docker run -it django-tutorial sh
```

-   this command tells docker to run a docker container based on the `django-tutorial` image we just created
-   the `-i` flag tells docker to run it interactively
-   the `-t` flag tells docker to use a "Allocate a pseudo-TTY". In other words let us interact with the container like a terminal
-   the `sh` command tells docker which command to run. In this case we're telling it to start a shell

-   After running the command we have shell access to the docker container
-   I like to think of this as sshing into the docker container
-   have a look around. Try running python

## Step 3 - Let's take a closer look at the Dockerfile

```Dockerfile
# pull official base image
FROM python:3.6

# set work directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt /usr/src/app/requirements.txt
RUN pip install -r requirements.txt

# copy entrypoint.sh
COPY ./entrypoint.sh /usr/src/app/entrypoint.sh

# copy project
COPY . /usr/src/app/

# run entrypoint.sh
ENTRYPOINT ["sh", "/usr/src/app/entrypoint.sh"]
```

-   PYTHONDONTWRITEBYTECODE: Prevents Python from writing pyc files to disc (equivalent to python -B option)
-   PYTHONUNBUFFERED: Prevents Python from buffering stdout and stderr (equivalent to python -u option)

## Step 4 - Setup a docker-compose file

-   Now we've setup one container to run our web server, we need another container
    to run our database (postgres)
-   Docker compose allows us to create multiple services in docker and connect them
-   Ultimately we should have one container running django, the other running postgres

#### Web service

-   We'll start by creating a service in our docker compose for our web service (django)

```yml
version: "3.7"

services:
    web:
        build: ./app
        command: python manage.py runserver 0.0.0.0:8000
        volumes:
            - ./app/:/usr/src/app/ # Volumes allow us to share the code
        ports:
            - 8000:8000
        env_file:
            - ./.env.dev
```

-   `web`: the name of our service (it can be anything you like)
-   `build`: Specifies the directory of which docker file to use
-   `command`: the command to run when the container starts in our case we want to start the django server
-   `ports`: This maps ports running in the container to ports running on your host
-   `volumes`: allow us to share code between our host machine and the container. This is how we can develop using docker

#### Database service

-   Now let's add database service
-   We're going to use the postgres base image from dockerhub (https://hub.docker.com/postgres)

```yml
  web:
    build: ./app
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./app/:/usr/src/app/
    ports:
      - 8000:8000
    env_file:
      - ./.env.dev
    depends_on:
      - db

  db:
    image: postgres:12.0-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_USER=hello_django
      - POSTGRES_PASSWORD=hello_django
      - POSTGRES_DB=hello_django_dev

volumes:
  postgres_data:

```

-   `image`: We don't have to build our own, we can use the postgres image directly from docker hub
-   `volumes`: For the database service we create a volume to save the data. If we didn't have this volume,
    all the data from our database would be wiped when our container stopped
-   `environment`: We can set the user, password, and database name according to the docs on the postgres dockerhub page

#### Web server entrypoint

-   We don't want to start django until our web server is running and migrations have been run
-   Let's create an `entrypoint.sh` for our django web server

```bash
#!/bin/sh

if [ "$DATABASE" = "postgres" ]
then
    echo "Waiting for postgres..."

    while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
    done

    echo "PostgreSQL started"
fi

python manage.py flush --no-input
python manage.py migrate

exec "$@"
```

-   **NOTE**: We can access the postgres host by using the service name "db"
-   We don't have to use the ip address of the machine. This is called **automatic service discovery**


- Now we can start the services.
- First navigate to the root directory then run:

```bash
docker-compose up --build
```

- If the solution worked, you should see the following in your terminal

```
Successfully built 949bad36172b
Successfully tagged docker-tutorial_web:latest
docker-tutorial_db_1 is up-to-date
docker-tutorial_web_1 is up-to-date
Attaching to docker-tutorial_db_1, docker-tutorial_web_1
web_1  | Operations to perform:
web_1  |   Apply all migrations: admin, auth, contenttypes, sessions
web_1  | Running migrations:
web_1  |   Applying contenttypes.0001_initial... OK
web_1  |   Applying auth.0001_initial... OK
web_1  |   Applying admin.0001_initial... OK
web_1  |   Applying admin.0002_logentry_remove_auto_add... OK
web_1  |   Applying admin.0003_logentry_add_action_flag_choices... OK
web_1  |   Applying contenttypes.0002_remove_content_type_name... OK
web_1  |   Applying auth.0002_alter_permission_name_max_length... OK
web_1  |   Applying auth.0003_alter_user_email_max_length... OK
web_1  |   Applying auth.0004_alter_user_username_opts... OK
web_1  |   Applying auth.0005_alter_user_last_login_null... OK
web_1  |   Applying auth.0006_require_contenttypes_0002... OK
web_1  |   Applying auth.0007_alter_validators_add_error_messages... OK
web_1  |   Applying auth.0008_alter_user_username_max_length... OK
web_1  |   Applying auth.0009_alter_user_last_name_max_length... OK
web_1  |   Applying auth.0010_alter_group_name_max_length... OK
web_1  |   Applying auth.0011_update_proxy_permissions... OK
web_1  |   Applying sessions.0001_initial... OK
web_1  | Watching for file changes with StatReloader
web_1  | Performing system checks...
web_1  |
web_1  | System check identified no issues (0 silenced).
web_1  | March 25, 2020 - 18:44:26
web_1  | Django version 2.2.6, using settings 'hello_django.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
web_1  | [25/Mar/2020 18:44:32] "GET / HTTP/1.1" 200 16348
web_1  | [25/Mar/2020 18:44:32] "GET /static/admin/css/fonts.css HTTP/1.1" 200 423
web_1  | [25/Mar/2020 18:44:33] "GET /static/admin/fonts/Roboto-Bold-webfont.woff HTTP/1.1" 200 86184
web_1  | [25/Mar/2020 18:44:33] "GET /static/admin/fonts/Roboto-Regular-webfont.woff HTTP/1.1" 200 85876
web_1  | [25/Mar/2020 18:44:33] "GET /static/admin/fonts/Roboto-Light-webfont.woff HTTP/1.1" 200 85692
db_1   | The files belonging to this database system will be owned by user "postgres".
```

- You can also navigate to http://localhost:8000 to see a running django web server


- lastly you can see all the running docker containers by running:

```bash
docker ps
```

- you should see the following
```
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                    NAMES
2545c9ed8b9a        docker-tutorial_web    "sh /usr/src/app/ent…"   37 minutes ago      Up 37 minutes       0.0.0.0:8000->8000/tcp   docker-tutorial_web_1
3969233cfb2c        postgres:12.0-alpine   "docker-entrypoint.s…"   42 minutes ago      Up 41 minutes       5432/tcp                 docker-tutorial_db_1
```


#### Docker compose commands

-   `docker-compose build` - builds the images
-   `docker-compose up` - Starts the already built images
-   `docker-compose up --build` - builds the images and starts the containers

#### Docker compose workflow

-   Typically, you'll have to run `docker-compose build` everytime you add a package
-   Anytime I pull a new branch I always run this command
-   If you haven't added any new packages, you can just run `docker-compose up`

## Step 5 docker swarm demo

-   Now that we have our docker compose file, we only have a few small steps left to prepare it for deployment to a swarm

1. All images must be built and on dockerhub (or another repository)
2. Add deployment labels (if needed)
    - Number of replicas
    - Which to deploy on managers etc.

## Pros/Cons

#### Developing with docker

-   Pros:
    -   Developing in docker. Everyone has the same environment
    -   Development environment is very similar to the production environment
-   Cons
    -   developing with docker is slower. Having to rebuild images everytime you add a package is a pain. **Note**: this isn't always needed but it's safe
    -   learning curve. Docker isn't easy and volumes/networks can be confusing

#### Deployment with docker swarm

-   Pros
    -   Deployment is much simpler than provisioning a VM and setting it up via bash scripts, terraform etc.
    -   Environment that's very similar to development and testing
    -   Flexible deployments/scaling. For example, I can scale up my web servers when needed without affecting other services
    -   Easy to add additional machines to the swarm
    -   Relatively easy to add services to the swarm. For example, adding elastic search for logging, redis for task queues, mailhog for email monitoring
-   Cons
    -   Deployments are more complex
    -   Multiple running containers can be more difficult to manage, setup, and debug
    -   Logging is more complex

## Additional learning resources
- Testdriven io has some excellent resources
    - https://testdriven.io/blog/dockerizing-django-with-postgres-gunicorn-and-nginx/
    - https://testdriven.io/blog/running-flask-on-docker-swarm/
- The docker docs
    - https://docs.docker.com/engine/swarm/
- https://dockerswarm.rocks
- A more in depth docker tutorial
    - https://docker-curriculum.com/
