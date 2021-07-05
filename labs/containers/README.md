# Running Containers

Containers are a virtual computing environment which Docker creates. When you run a container it typically runs a single process, and you can have that process running in different ways.

## CLI Reference

- [Docker command line reference](https://docs.docker.com/engine/reference/commandline/cli/)
- [Container commands](https://docs.docker.com/engine/reference/commandline/container/)
- [docker run](https://docs.docker.com/engine/reference/commandline/run/)

<details>
  <summary>CLI overview</summary>

The Docker command line sends instructions to the Docker API. 

Commands are grouped by types of object (e.g. containers and networks). You can always print help to list the available commands and the options for a specific command:

```
docker

docker container --help

docker container run --help
```
</details><br/>

## Running a one-off container

Docker containers run as long as the process inside the container keeps running. When the process finishes, the container exits.

You can run a simple container to run a single Linux command:

```
docker run alpine hostname
```

- lots of Docker commands have aliases - `docker run` is the same as `docker container run`.

> You'll see a lot of output when you run this - Docker pulls the Alpine Linux container image so it can run locally, starts a container, runs the hostname command and prints the output.

📋 Run another container with the same hostname command.

<details>
  <summary>Not sure how?</summary>

```
docker run alpine hostname
```

</details><br/>

> The container image doesn't get pulled this time, but the output is a different random string.

Those containers ran a single command - when the process completed, the container exited.

📋 Print a list of all your containers.

<details>
  <summary>Not sure how?</summary>

```
# print running containers:
docker container ls

# or use ps; the a flag shows all statuses
docker ps -a
```

</details><br/>

> You have two containers, both in the exited state. The container IDs match the hostname output.

Those containers didn't do anything useful, but  one-off containers can be good for automation tasks, they can be packaged with all the tools and scripts they need.

Try a container which generates a security certiicate you can use for HTTPS in dev environments:

```
docker run -it kiamol/ch15-cert-generator
```

> The `-it` flag attaches your terminal, so you can send commands to the container. When you see the output `Certs generated`, exit the container by pressing Ctrl-C / Cmd-C.

📋 The cert-generator container has exited; print out the logs from the process.

<details>
  <summary>Not sure how?</summary>

```
docker container ls -a  # to find the container ID

docker container logs <container-id>
```

</details><br/>


> Container logs are the output from the container process - Docker saves them when the container exits.

## Running an interactive container

One-off containers run and then stop. You can run a long-running process in a container and connect your terminal to the container's terminal.

This is like connecting to a remote machine - any commands you run actually run inside the container:

```
docker run -it alpine
```

- the `-it` flag means runs interactively, with the local terminal attached

Now you're connected to the container, you can explore the environment:

```
ls /
whoami
hostname
cat /etc/os-release

exit
```

> You can see the container behaves like a full Linux machine, running Alpine Linux.

Linux OS containers for Docker usually have the bare-minimum toolset installed.

Ubuntu doesn't have curl installed, but the container runs as the root user so you have permissions to install anything you need:

```
docker run -it ubuntu

curl https://dockerfun.courselabs.co   # command not found

apt-get install -y curl  # this doesn't work - there's no install cache

apt-get update  # update the cache

apt-get install -y curl  # now this works

curl https://dockerfun.courselabs.co

exit
```

> The changes you've made are only for that one container.

📋 Run another Ubuntu container and see if it can use curl.

<details>
  <summary>Not sure how?</summary>

```
# you can do this as a one-off container - it won't work:
docker run ubuntu bash -c 'curl https://dockerfun.courselabs.co'
```

</details><br/>

Interactive containers can do fun things with the screen - this is useful to impress people with your hacking skills :)

```
docker run -it sixeyed/hollywood

# Ctrl-C / Cmd-C to stop the process

# and exit the container
exit
```

## Running a background container

Interactive containers are great for testing commands and exploring the container setup, but mostly you'll want to run _detached_ containers.

These run in the background, so the container keeps running until the container process exits, or you stop the container.

You'll use background containers for web servers, batch processes, databases message queues and any other server process.

```
docker run -d -P --name nginx1 nginx:alpine
```

- the `-d` flag runs a detached background container
- `-P` publishes network ports so you can send traffic into the container
- `--name` gives the container a name so you can work with it in other commands

> You now have a simple web server running in the container called `nginx1`.

📋 Find out which port the container is listening on, and try browsing to the web server.

<details>
  <summary>Not sure how?</summary>

```
# print the container's port mapping
docker container port nginx1

# browse to the port with curl or your browser
curl localhost:<container-port>
```

</details><br/>

> Docker listens for incoming traffic on your machine's network port and forwards it to the container.

## Lab

We've run containers using Alpine Linux and Ubuntu, and also with Nginx installed. They're all public packages available on [Docker Hub](https://hub.docker.com).

In this lab you'll work with Java containers:

- find a package on Docker Hub which you can use to run a Java app
- run a Java container and confirm which version of Java is installed using the `java -version` command
- now find a *small* image for Java 8, with just the JRE runtime installed

> Stuck? Try [hints](hints.md) or check the [solution](solution.md).

___
## **EXTRA** Multiple containers

<details>
  <summary>Publishing different ports</summary>

Containers are isolated compute environments, you can run multiple containers from the same package - they all behave in the same way, but they're separate.

Try running a container with a simple web server:

```
docker run -d -p 8080:80 sixeyed/whoami:21.04

curl localhost:8080
```

The `-p` flag - lowercase p - publishes a specific port, so here Docker listens on port 8080 and sends traffic into the container on port 80.

Ports are single-use resources, if you repeat the command you'll get a failure.

📋 Run some more containers from the same image, using different ports.

<details>
  <summary>Not sure how?</summary>

```
docker run -d -p 8081:80 sixeyed/whoami:21.04
docker run -d -p 8082:80 sixeyed/whoami:21.04
docker run -d -p 8083:80 sixeyed/whoami:21.04
```

</details><br/>

Check the running containers: 

```
docker ps
```

They each listen on different ports, so you can see the response from each container:

```
curl localhost:8081
curl localhost:8082
curl localhost:8083
```

</details><br/>

___
## Cleanup

Cleanup by removing all containers:

```
docker rm -f $(docker ps -aq)
```