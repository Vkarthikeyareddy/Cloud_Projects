# Part 5: Container Networking

Container networking allows us to interact with our isolated containers. Through container networking, we can have containers communicate with each other or to non-containerized workloads. 

For example, if we were running a web application server and we were expecting traffic on ports 443 (for https traffic) and port 80 (for http traffic), we would need to make sure that the webserver container can accept traffic on those ports. 

There are some key models to help you along with your container networking journey so that your applications can communicate. 

# Networking Models

## None

- The simplest mode of networking is a loopback interface in which the container does not communicate with an external network. The container has a network stack, but no external connection.
    - A loopback interface is a specialized network interface that is only available to the local host, e.g., it’s only accessible at 127.0.0.1
    - You can use this model to test containers, assign containers without external communication, and stage containers for future network connections.

## Bridge

- A bridge network is an internal host network that enables communication between containers on the same host. Note that containers cannot be accessed from outside the host.
- Bridge networking uses iptables for network address translation (NAT) and port mapping to provide single-host networking. This is the default Docker network type (docker0)

## Host

- A host model allows a container to share its network namespace with the host, enabling high-speed networking. This grants the container access to all of the host’s network interfaces.
- A host network eliminates the need for NAT, but it can lead to port conflicts because the container has access to the host’s network interfaces.

## Overlays

- Overlays enable communication across hosts using network tunnels, allowing containers on different hosts to behave as though they were on a single machine. Containers connected to different overlay networks cannot communicate.
    - Docker uses the virtual extensible local area network (VXLAN).
    - Each cloud provider tunnel type creates a dedicated route for each VPC or account.
- Network overlays ensure that containers connected to two different overlay networks are isolated from each other and cannot communicate over the local bridge.

## Underlays

- Underlays expose host interfaces to the VMs or containers running on the host. They are suited for handling on-prem workloads, and workloads requiring higher levels of security and compliance. Underlays remove the need for port-mapping, making them more efficient than bridges.

# Common Docker networking techniques

## bridge

Users can create custom networks and connect multiple containers to the same network. Once a container is connected to a network, they can communicate with each other using container IP addresses or container names. 

For example, the following command will create a network called my-net:

```yaml
docker network create -d bridge my-net
```

my-net is a bridge network, as defined by the use of `-d bridge`. -d defines the network driver (the default in Docker is a bridge network, so if we left out `-d bridge` then we would still create a bridge network called `my-net` 

Now, if we run the following command to create a new container, it will run on the my-net bridge network:

```yaml
docker run --network=my-net -itd --name=container2 alpine
```

<aside>
💡

-it is short for `—-interactive` + `—-tty` . This will create the container and return an interactive shell directly inside the container.

-d is short for `--detatch` , which means your container will run in the background.

</aside>

You can also attach a container to another container’s networking stack directly, by using the `--network container:<name or ID>`  flag format. 

Say we wanted to create a redis container and bind it to localhost. We would run the following command:

```yaml
docker run -d --name redis example/redis --bind 127.0.0.1
```

Then, we want to run the redis-cli. In this case, we want the redis cli to talk  to the redis database container. We would configure the redis cli container to run as follows:

```yaml
docker run --rm -it --network container:redis example/redis-cli -h 127.0.0.1
```

## publish

When you create a container using `docker run` , containers on bridge networks don’t expose any ports to the host, to other containers, or to an external network. Using the `--publish` or `-p` flags makes ports available to services outside the bridge network. The `-p` flag creates a firewall rule in the host, mapping a container port to a port on the Docker host to the outside world.

The `-p` flag (short for `--publish`) creates a port mapping between the host and the container. The `-p` flag takes a string value in the format of `HOST:CONTAINER`, where `HOST` is the address on the host, and `CONTAINER` is the port on the container. 

Take the following example. Say we have a container image we want to run called python-docker. It’s a web application that expects traffic on port 3000. We’re going to spin it up and test it on our local machine, so that it’s accessible at 127.0.0.1 Without using `-p` , 

```yaml
docker run -p 3000:3000 python-docker
```

The command publishes the container's port 3000 to `127.0.0.1:3000` (`localhost:3000`) on the host. Without the port mapping, you wouldn't be able to access the application from the host.

Other examples:

`-p 8080:80` : Map port 8080 on the host to port 80 in the container

`-p 192.168.1.100:80808:80` : Map port 8080 on the host IP 192.168.1.100 to port 80 in the container

`-p 8080:80/udp` : Map port 8080 on the host to UDP port 80 in the container.

Important note:

- When you publish a container port, it’s not only accessible to the host but to the outside world as well.
- If you include the [localhost](http://localhost) IP address (127.0.0.1 or `::1` ) with the publish flag, only the host and its containers can access the published container port.