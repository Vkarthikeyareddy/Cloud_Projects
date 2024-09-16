# Part 1: Docker First Steps

Let’s first make sure that Docker is installed and is working correctly. You should have installed Docker desktop in Assignment 1 (Getting Started.) 

Verify that you can run Docker by typing the following

```yaml
docker run hello-world
```

You will get a message printed to the console saying:

```yaml
Hello from Docker.
This message shows that your installation appears to be working correctly.
...
```

We’ll be using the official [Alpine Linux](https://hub.docker.com/_/alpine) image from Docker Hub for the next part of this tutorial. [Alpine Linux](https://alpinelinux.org/about/) is a very lightweight Linux distro- a containerized implementation of Alpine linux requires no more than 8MB, and provides a full-fledged Linux environment.  

## Part 1: docker pull

First, we will pull the Alpine Linux image from Docker Hub. Open a command prompt or terminal, and run:

```bash
docker pull alpine
```

If you get a permission denied error, try the following:

Make sure that Docker is running (when you launch Docker Desktop, the Docker daemon will automatically start). To start the Docker daemon without launching Docker Desktop, run

```bash
dockerd
```

or 

```bash
sudo systemctl start docker
```

The pull command downloads the image from Docker Hub. You can see a list of local images by using the `docker images` command:

```bash
docker images
```

```bash
REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
alpine                 latest    c157a85ed455   2 days ago     8.83MB
```

Note the TAG column: this means that we’ve pulled the latest published Alpine image. 

## Part 2: docker run

Now let’s run our container. We’ll use the `docker run` command to launch a container from the image we downloaded. Remember that an image is like a blueprint for a container. 

```bash
docker run alpine
```

When we call docker run the Docker client finds the image (e.g. Alpine), creates the container and then runs a command in that container. We didn't provide a command to our docker run call, so the container booted up, ran an empty command and then exited. 

Let’s try passing a command to docker run:

```bash
docker run alpine echo "hello from alpine"
```

The Docker client ran the `echo` command in our alpine container and then exited. Think about what the steps would be to echo “hello from alpine” if we ran this as a VM:

We would have to:

1. Locate the VM image
2. Create a VM using a CLI, API, or the cloud console
3. Wait for the VM to be fully created with an IP address
4. SSH to the VM
5. type `echo "hello from alpine` 

That VM would also cost us around $4/month. 

## Part 3: docker ps

The `docker ps` command will show all of the Docker processes that are running.  Go ahead and run:

```bash
docker ps
```

We don’t see any containers running because the alpine container launched, ran its command, and then exited.

A helpful command is to run `docker ps -a` . This will show a list of all of the containers we’ve run, as well as their state. The STATUS column will show that you stopped a container a few minutes ago.

Run:

```bash
docker ps -a
```

CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS                     
6efb426a1d5d   alpine                 "echo 'hello from al…"   4 minutes ago   Exited (0) 4 minutes ago             
427b574c2949   alpine                 "/bin/sh"                6 minutes ago   Exited (0) 6 minutes ago             

Note the STATUS column show that the containers exited. You can also see the COMMAND that was run. Our first container ran /bin/sh (a command that launches the system shell) , and our second container ran our “echo ‘hello from alpine’” command. 

Let’s try running more than one command at a time:

```yaml
docker run -it alpine sh
```

passing the `-it` flag provides us with an interactive tty (**T**ele**Ty**pewriter), meaning that you can interact with the running container! 

Now you can type common Linux commands. Launching your alpine container is a great way of trying out commands on Linux before you run them IRL. 

Once you’re done, you can type `exit`  to exit the Alpine shell.

## Part 4: docker rm

Let’s use the `docker ps -a` command again to see all of the containers we used so far. 

You should see three alpine containers. Leaving stray containers on your machine can use up disk space.You can clean up your containers as you run them by using the `docker rm` command. Copy the container IDs from the ps -a command and pass them to the `docker rm` command:

```yaml
docker rm <container ID here>
```

Now run `docker ps -a` again and you should see that those containers are no longer there.

You can also use the `docker container prune` command to remove any containers that have been exited.