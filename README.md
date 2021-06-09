# PERSONAL DJANGO WEBSITE DEVELOPMENT

I am going to document the steps I take to develp my personal website here so
that I have record of them, and can reproduce my results in the future.


## TODO
- [ ] Get a Django project up and running in Docker


# MY PROCESS

For the starting of this project I used the following [video tutorial](https://www.youtube.com/watch?v=KaSJMDo-aPs)
to get a django project up and running in Docker. This [website](https://www.codingforentrepreneurs.com/blog/django-on-docker-a-simple-introduction)
also has the same instructions written out.

### 1. First steps will be to make a directory to store the project and then we will
be setting up a virtual environment with pipenv. (last night I had trouble
getting a virtual environment to work on Ubuntu for windows so I will see if
this works better.)

```bash
$ mkdir django-michaels-website
$ cd django-michaels-website
```
I did not have pipenv on my machine so I had to run the following commands
```bash
$ sudo pip3 install pipenv
```
In the tutorial they install django==2.2.4 gunicorn --python 3.6 but I want to
be running the most up to date packages so I changed these values.
```bash
$ pipenv --python python3
$ pipenv install Django==3.1.1 gunicorn --python 3.8
$ pipenv shell
```
[pipenv documentation](https://pypi.org/project/pipenv/). pipenv is supposed
to simplify creating virutal environments so instead of using requirements.txt
it uses a Pipfile

After building the virtual environment I got the following instructions from
pipenv.

To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.

### 2. Create a Django Project
```bash
django-admin startproject michaelhome .
```

### 3. Local DB & Migrations
```bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

### 4. Update Django Settings.py
```python
# DEBUG can be True/False or 1/0
DEBUG = int(os.environ.get('DEBUG', default=1))
```


### 5. Create .env
```bash
DEBUG=1
```

### 6. Test gunicorn
```bash
gunicorn michaelhome.wsgi:application --bind 0.0.0.0:8000
```
This is what we can run in lieu of
```bash
python manage.py runserver
```
At this point I could develop the whole application. I want to get everything
up and running and then I will develop from there.

## Docker File, Image, & Container

### 1. Create Dockerfile

In the root directory for the project run the bash command:
```bash
$ touch Dockerfile
```
this will create the Dockerfile.

Open it in the text editor and add the following code.
```Docker
# Base Image
FROM python:3.8

# create and set working directory
RUN mkdir /app
WORKDIR /app

# Add current directory code to working directory
ADD . /app/

# set default environment variables
ENV PYTHONUNBUFFERED 1
ENV LANG C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

# set project environment variables
# grab these via Python's os.environ
# these are 100% optional here
ENV PORT=8888

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        tzdata \
        python3-setuptools \
        python3-pip \
        python3-dev \
        python3-venv \
        git \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


# install environment dependencies
RUN pip3 install --upgrade pip
RUN pip3 install pipenv

# Install project dependencies
RUN pipenv install --skip-lock --system --dev

EXPOSE 8888
CMD gunicorn michaelhome.wsgi:application --bind 0.0.0.0:$PORT
```

### 2. Build Docker Image
```bash
$ docker build -t django-michaels-website -f Dockerfile .
```
The `-t` flag stands for tag and we use it to add a name to the image we are
building

### 3. Run your Container
```bash
docker run -it -p 80:8888 django-michaels-website
```
The `-p` flag stands for port.

With the above configuration for running it you will just have to type in
localhost to be able to see the website. But if you want to be able to use the
port, e.g. `localhost:8888` Then you have to switch the 80 to 8888 as well.

The 80 is there because generally websites will be exposed on port 80. If it is
on port 80 you don't need to show the port in the url.

If we want the container to run in the background we just need to add the `-d`
flag between `-it` and `-p`.
```bash
docker run -it -d -p 8888:8888 django-michaels-website
```

To stop the container from running in the background run the following command:
```bash
$ docker ps
```
this will show you the docker images that are running. You will want to take
the id given in the output and then run the command.
```bash
$ docker stop <id>
```

### 4. Clean Up
List running containers
```bash
$ docker ps -a
```

Stop your container
```bash
$ docker stop <id>
$ docker rm <id>
```

The `rm` removes the docker images from your images. To view all the docker
images on your computer run the command
```bash
docker images
```

The below commands removes other objects associated with the image.
```bash
$ docker image rm django-michaels-website --force
```

Docker provides a single command that will clean up any resources — images, containers, volumes, and networks — that are dangling (not associated with a container):
```bash
$ docker system prune
```

To additionally remove any stopped containers and all unused images (not just dangling images), add the -a flag to the command:
```bash
$ docker system prune -a
```
The following link gives more information on cleaning things up.
[Cleaning up Docker](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes)


## References that may be Useful

* [Deploy Docker to Production on Heroku](https://www.codingforentrepreneurs.com/blog/django-docker-production-heroku/)

## References used
* [PostgreSQL DB with Django Docker Application](https://www.youtube.com/watch?v=610jg8bK0I8)
Go through this tutorial to see how I can connect the docker application to my
local postgres database.
