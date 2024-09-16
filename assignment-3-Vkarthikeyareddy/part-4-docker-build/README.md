# Part 4: Docker Build

The next part to containerizing our application is to build the image. We accomplish this using the `docker build` command. 

`docker build` uses the Dockerfile to build a new image. The instructions from the Dockerfile in your application are going to add layers to your image- whenever you use the `RUN` instruction in your Dockerfile, you’re adding to your layers. 

The basic command for `docker build` is 

```bash
docker build -t <image-name-here> <where-to-find-the-dockerfile> 
```

For example, if we’re building an image with the name “my-first-image” and we’re running the `docker build` command from the same directory that our Dockerfile lives in, we would run:

```bash
docker build -t my-first-image .
```

Note the `.` at the end of the command. That’s telling `docker build` that the Dockerfile is in the current directory. 

## Docker build flags

`docker build` has several flags you can use to specify how your image should be built. 

the `-t` flag is commonly used- it tags your image with a human-readable name. Passing `-t` will mean that you can refer to your image by its name later on. 

## Multi-platform builds

Recall from lectures 2 and 4, that different architectures will require different image builds. For example, we need to create images that can be run on `linux/amd64` , `linux/arm64`  and `windows/amd64` . Containers share the host kernel, which means that the code that’s running inside the container must be compatible with the host’s architecture. 

Multi-platform builds solve this problem by packaging multiple variants of the same application into a single image. When you push a multi-platform image to a registry, the registry will store the manifest list and all of the individual manifests. When you pull the image, the registry returns the manifest list, and Docker automatically selects the correct variant based on the host’s architecture. 

To enable multi-platform builds, you’ll need to go to Docker Desktop, select “Settings”, and select “Use containerd for pulling and storing images” in the General tab. 

To build a multi-platform image, use the `--platform` flag to define the target platforms for the build output:

```bash
docker build --platform linux/amd64, linux/arm64
```

For example, let’s build a simple multi-platform image:

1. In the this directory (e.g., you’ve cloned Assignment 3, navigate into it and then into Part 4: docker build), create a Dockerfile. For example: 

```bash
cd part4
touch Dockerfile
```

1. Add the following to your Dockerfile:

```yaml
# syntax=docker/dockerfile:1
FROM alpine
RUN uname -m > /arch
```

1. Build the image for `linux/amd64` and `linux/arm64` 

```bash
docker build --platform linux/amd64,linux/arm64 -t multi-platform .
```

1. Run the image and print the architecture:

```bash
docker run --rm multi-platform cat /arch
```

If you’re running on an x86-64 machine, you should see `x86_64` . If you’re running on an ARM machine, you should see `aarch64` . 

Once your image is built, you will be able to see it in your Docker Desktop UI. Go to the “Builds” tab to see any images that you’ve built.