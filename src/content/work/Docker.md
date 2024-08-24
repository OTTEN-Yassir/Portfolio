---
title: Docker Report
publishDate: 2024-02-23 23:29:30
img: /assets/IMG-2-Report-Docker.png
img_alt: OTTEN Yassir
description: |
  A comprehensive report on the introduction to Docker, detailing its setup, usage, and the practical applications demonstrated during the course. This document covers the basics of Docker, its architecture, and step-by-step guides on container creation, image management, and network configuration.
tags:
  - Docker
  - Network and Services
---

## Introduction

During AIR5, we acquired an in-depth understanding of the use, manipulation, and configuration of Docker. First, I will describe the fundamentals of Docker and its utility. Then, I will recap the various manipulations performed in the laboratory to explore the different stages of Docker implementation.  
 [link](https://github.com/OTTEN-Yassir/IT-ICT-Reports/blob/main/DOCKER.md) to the GitHub repository

## What is Docker?

Docker is an open-source platform that allows the creation, distribution, and execution of applications in lightweight, isolated containers (based on Linux LXC virtualization). These containers encapsulate the application along with all its dependencies, providing portability and execution consistency, regardless of the hosting environment. Docker's architecture is based on images and containers. Unlike virtualization (VM), containers rely on the existing hardware and OS of the host machine.

Docker operates on a client-server architecture with the Docker daemon (Docker engine) and Docker client. Both can run on the same machine.

- **Image**: A ready-to-use template with instructions for creating containers. The image can be built from a Dockerfile, an existing image, or fetched from a Docker registry (Docker Hub, a public image repository).

- **Container**: The environment running for the image, providing a level of abstraction.

![container](/assets/IMG-1-Report-Docker.png)

### Process

1. **Creation (build) / Import (pull)** of the image, either from Docker Hub or a Dockerfile.
2. Once the image is created/imported, **create (run)** the containers.
3. Later, you can convert this container into an image itself (**commit**).
4. If needed, you can also store it on Docker Hub (**push**).

![process](/assets/IMG-2-Report-Docker.png)

When you install Docker on your system, it creates a default network bridge interface called `docker0`. This bridge interface allows Docker containers to communicate with each other and the host network by assigning them an IP address within the same network. It also isolates the traffic between containers while connecting them to the host network and beyond. It has characteristics of a switch.

### Example of Intra-Container Communication:

1. On a Linux machine, I opened 3 terminals. After installing Docker and verifying it's running with the command 

![status](/assets/IMG-3-Report-Docker.png)

1. I pulled the CentOS image from Docker Hub.

![pull](/assets/IMG-4-Report-Docker.png)

![list images](/assets/IMG-5-Report-Docker.png)

3. I created a container, specifically a bash shell.

![run](/assets/IMG-6-Report-Docker.png)

4. I repeated step 3 in another terminal window to have two containers running in parallel.
    
5. In a third terminal, I ran the command `ip a` to see the IP addresses my machine has. I noticed that there is a `docker0` interface with the IP `172.17.0.1/16`, which will serve as the bridge between the containers. I also saw 2 other `veth` interfaces with different IDs representing my two running containers.

![ip](/assets/IMG-7-Report-Docker.png)

6. In each container, I executed the command `ip a`. I noticed that an IP address in the same subnet as the `docker0` bridge was assigned to them: `172.17.0.2/16` and `172.17.0.3/16`.

![ip](/assets/IMG-8-Report-Docker.png)

![ip](/assets/IMG-9-Report-Docker.png)

7. I tried to ping from one container to the IP address of the other running container.

![ping](/assets/IMG-10-Report-Docker.png)

8. From my third terminal, I attempted to listen to the traffic on the `docker0` interface using the `tcpdump` command. The ping packets from both containers were clearly visible in the traffic.

![tcpdump](/assets/IMG-11-Report-Docker.png)

## Dockerfile?

A Dockerfile is a text script that contains a series of instructions to build a Docker image. It is essential for defining dependencies, environment configuration, and the execution steps necessary for an application to run properly in a Docker container. It also allows for customization.

### To Create Your Own Image:

1. You need to edit a Dockerfile (case-sensitive).

![dockerfile](/assets/IMG-12-Report-Docker.png)

    - FROM: Specifies the base image on which to build your image.
    - WORKDIR: Specifies the working directory for the next instructions.
    - COPY: Copies the `hello.py` file into the working directory.
    - CMD: Specifies the default command that the container will execute when launched.
2. Create your `hello.py` file in the same directory as your Dockerfile.

![hello.py](/assets/IMG-13-Report-Docker.png)

![hello.py](/assets/IMG-14-Report-Docker.png)

1. Now build your image using the command `docker build -t myContainerName .` (`-t` for tag, `.` because the file is in the current directory).

![build](/assets/IMG-15-Report-Docker.png)

1. Launch your container with the command `docker run myImage`.

![run](/assets/IMG-16-Report-Docker.png)

### Useful Docker Commands:

- `docker run <options> image-name`: Launch a container.
- `docker ps`: List running containers.
- `docker ps -a`: List containers, including stopped ones.
- `docker stop container-name`: Stop a container.
- `docker start container-name`: Start a stopped container.
- `docker rm container-name`: Remove a container.
- `docker exec -it container-name sh`: Interactive shell within a running container.
- `docker images`: List local images.
- `docker pull image-name`: Download an image from Docker Hub.
- `docker rmi image-name`: Remove an image.
- `docker build -t image-name path-to-context`: Build an image from a Dockerfile.
- `docker info`: Get information about the Docker daemon.

Each time a container is launched, it is assigned a random name for identification. Other Docker features exist, and we could not explore everything within the scope of the course. Docker Swarm with Docker-Compose was used during our transversal project.