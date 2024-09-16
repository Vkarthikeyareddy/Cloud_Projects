# Part 3: Dockerfile syntax

So far, we’ve pulled images from Docker Hub and run those images as-is locally. When we’re ready to containerize our own applications, we need to define how the image will be built. 

We do this through using Dockerfiles. 

## Syntax and conventions

- The syntax for Dockerfiles always follows this convention:

```docker
INSTRUCTION argument
```

- Although the instruction is not case sensitive, the convention is to use UPPERCASE for instructions.
- Docker runs instructions in order from top to bottom.
- Comments in Dockerfiles can be created by starting a line with `#`
- Environment variables are notated in Dockerfiles using either `$variable_name` or `${variable_name}`

## Base image

Recall that we always start our Dockerfile with a **base image**. The image that you create that is purpose-built to run your application is an extension of a base image, meaning you’re adding layers on top of that base image. There are several places you may wish to pull a base image from. You can use [Docker Official base images](https://docs.docker.com/trusted-content/official-images/) , which are hardened, battle-tested, and efficient images. Additionally, there are [Verified Publisher Images](https://hub.docker.com/search?q=&image_filter=store&_gl=1*nyr2un*_ga*MTc4MTg5NjI3My4xNzE4MDY4MDIz*_ga_XJWPQMJYHQ*MTcyNTg0MzA3Ni4xMS4xLjE3MjU4NDQxMzcuNDAuMC4w). These are verified and published by trusted developers. They will often include specialized software required for using specific tools. 

Unless you are pulling a base image from your employer’s private registry, you should always start with a Docker Official or Verified Publisher image! 

The base image is declared by using the `FROM` instruction in your Dockerfile. This will always be the first instruction. For example:

```docker
FROM python:3.10-alpine
```

## Working directory

The working directory is specified by using the `WORKDIR` instruction. Let’s say we’re building a Flask web app. Locally, our file structure may look like this:

```docker
.
├── app
    ├── requirements.txt
    └── app.py
```

Once we add our Dockerfile, our directory will look like this

```docker
.
├── app
    ├── Dockerfile
    ├── requirements.txt
    └── app.py
```

When we create our image, we want to set a working directory that will be the basis for our commands to execute in. Let’s append to our Dockerfile to set the working directory using the `WORKDIR` instruction.

```docker
FROM python:3.10-alpine

WORKDIR /app
```

## Copying files into the image

Now that we have a working directory set, we want to make sure that the image is going to include any files that it needs to run. First, let’s copy the requirements.txt file. Recall that our local project directory looks like this:

```docker
.
├── app
    ├── Dockerfile
    ├── requirements.txt
    └── app.py
```

We can copy our requirements.txt file into our working directory by using the `COPY` instruction. Let’s append to our Dockerfile:

```docker
FROM python:3.10-alpine

WORKDIR /app
COPY requirements.txt /app
```

`COPY` ’s first argument is the source, and the second argument is the destination. So in our case, we are copying requirements.txt (the source) into /app (the destination.) 

Why don’t we have to specify that requirements.txt is in the /app folder in our local project? Because the path will be relative to where the Dockerfile lives! If our Dockerfile lived here instead:

```docker
.
├── Dockerfile
├── app
    ├── requirements.txt
    └── app.py
```

then we would need our `COPY`  instruction to go to our local /app directory and then copy the files from there into the /app working directory:

```docker
COPY app/requirements.txt /app
```

## Build layers in our Docker image.

Our next step is to install the dependencies from requirements.txt. We can do that using the `RUN` instruction. Let’s append the `RUN` instruction to our Dockerfile:

```docker
FROM python:3.10-alpine

WORKDIR /app
COPY requirements.txt /app
RUN pip3 install -r requirements.txt
```

The `RUN` instruction is used in Dockerfiles to execute commands that build and configure the Docker image. These commands are executed during the image build process, and each `RUN` instruction creates a new layer in the Docker image. 

For example, if you create an image that requires specific software or libraries installed, you would use `RUN` to execute the necessary installation commands.

The `RUN` instruction will run whatever command comes after it. In our case, we are going to use pip3 to install all of the dependencies in requirements.txt. 

At this point, our requirements will have been installed, and we can copy everything else in our project directory into the image. Let’s append to our Dockerfile again:

```docker
FROM python:3.10-alpine

WORKDIR /app
COPY requirements.txt /app
RUN pip3 install -r requirements.txt

COPY . /app
```

The dot notation here `.`  means to copy everything in the directory that our Dockerfile lives in into /app. In this case, the only additional thing that will be copied is app.py, but if other files lived in that directory we would want to include them. 

## Set an Entry Point

The `ENTRYPOINT` instruction sets the default executable for the container. Any arguments that are supplied to the docker run command are *appended* to the `ENTRYPOINT` . 

In Part 1- Docker First Steps, the command that ran when we didn’t add any arguments to the `docker run` command was ‘/bin/sh’ . (You can verify this by checking out the alpine image in Docker Desktop.) So when do we use `ENTRYPOINT` and when do we use `CMD` ? 

We want to use `ENTRYPOINT` when we need our container to always run the same base command, and we want to allow users to append additional commands at the end. One caveat is that `ENTRYPOINT` can be overridden on the `docker run` command line by supplying the `--entrypoint` flag. 

`ENTRYPOINT`  is particularly useful for turning a container into a standalone executable. For example, suppose you are packaging a custom script that requires arguments (e.g., `“my_script extra_args”`). In that case, you can use

```docker
ENTRYPOINT ["my_script"]
```

to always run the script process `“my_script”` and then allow the image users to specify the `“extra_args”` on the `docker run` command line. 

The `CMD` instruction can be used to provide default arguments to an `ENTRYPOINT` if it is specified in the exec form. This setup allows the entry point to be the main executable and `CMD` to specify additional arguments that can be overridden by the user.

In our case, we always want to start our script with `python3` and `app.py` can be run with additional arguments if needed. 

Let’s append to our Dockerfile:

```docker
FROM python:3.10-alpine

WORKDIR /app
COPY requirements.txt /app
RUN pip3 install -r requirements.txt

COPY . /app

ENTRYPOINT ["python3"]
CMD ["app.py"]
```

Keep these steps in mind when you’re creating your Dockerfile for Assignment 3. 

## Best Practices

### Users

We want to create a non-privileged user for our container to run as. By default, Docker containers run as privileged (e.g., root) users. Privileged users inherently have sudo access in the containerized environment. Attackers may utilize this fact to escape the containerized environment and gain access to the underlying host machine. For that reason, we use the `USER` instruction to have our container run as a non-privileged user. 

```docker
ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser
```

The `ARG` instruction can be thought of as a local variable that can be used in other parts of the Dockerfile. 

In this case, we use `ARG UID=10001` to create a Linux user ID for our container to run as. 

The `RUN` instruction tells our container to add a user, with no password, with no user information, a null home directory, a fake login shell, and the UID 10001. The final argument is the name of the user, in this case it will be “appuser”.

## Best Practices for Python

Every language will have some quirks when working with containers. Python has a few gotchas to look out for. Think about adding these instructions when creating containers for Python applications. 

PID 1: When a python application runs outside of a container, it will have a PID that’s somewhere between 2 and [max number of processes that can be run on the OS]. In containers, python will often run as PID1. That makes it very difficult to kill a python process, particularly when it’s running as part of a larger deployment system on a host that’s shared with other containers. Consider adding this line to make python not run as PID1: 

### References:

[https://www.docker.com/blog/docker-best-practices-choosing-between-run-cmd-and-entrypoint/](https://www.docker.com/blog/docker-best-practices-choosing-between-run-cmd-and-entrypoint/)