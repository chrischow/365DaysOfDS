---
layout: page
title: Docker Cheatsheet
background: '/img/posts/computing.jpg'
---

## What are Containers?

### Problems
* Different technologies in developing application stack e.g.
    * NodeJS as web server
    * MongoDB for database
    * Redis for messaging
    * Ansible for Orchestration
* Issues in developing, building, and shipping application:
    * Compatibility/dependency: The Matrix from Hell
        * Compatibility with OS
        * Compatibility of libraries and dependencies (e.g. different tools require different versions)
    * Take very long to set up
    * Different dev/test/production environments
* Docker:
    * Each tool can run in separate containers with libraries and dependencies

### Defining Containers
* Isolated environments with their own processes, networks and mounts
* Kind of like VMs, but they share the same OS kernel
* Docker is a high-level tool to set up containers
* Docker's main purpose is to package and containerise applications and ship and run them anywhere

### OS
* Comprise OS kernel and set of software
    * OS interacts with hardware
    * Software makes OSes different
* When Docker shares kernel, it is compatible with any OS as long as it's the same kernel e.g. Linux

### Virtual Machines vs. Containers
* Each VM has its own OS + libraries and dependencies (libs/deps), while Docker is installed in one OS, with libs/deps in each container
* VMs have higher utilisation due to multiple OSs running, while Docker containers consume less
* VMs consume higher disk space due to OS + libs/deps, while Docker containers only have libs/deps
* VMs take longer to boot up due to more software than Docker containers
* VMs don't rely on underlying OS so no shared resources, but Docker containers have shared resources

### Docker Images
* Many applications have a Docker image
* Just need to go `docker run nodejs`

### Container vs. Image
* Image:
    * A template or package
    * Used to create one or more containers
* Containers:
    * Running instances of images
    * Isolated with their own environments and processes

## Docker Commands

### `run`
* Run a container from an image
* If image is not present, it'll go to Docker Hub to download the image

#### Tags
* Below, the requested version `4.0` is the tag.
* Without the tag, it uses the default `latest` tag

```bash
docker run redis:4.0
```

#### Interactive Mode
The `-i` flag means interactive mode to allow inputs.

#### Pseudo Terminal
The `-t` flag attaches to a pseudo terminal to allow prompts. Can be used together with interactive mode: `-it`

#### Check Version of Image OS
```bash
docker run ubuntu cat /etc/*release*
```

#### Naming
Use the name tag:

```bash
docker run --name NameHere ImageName
```

### `ps`
* Lists running containers and some info about them

### `stop`
* Provide container ID or name

### `rm`
* Remove stopped or exited container entirely, even from "exited" state
* Helps to reclaim disk space

### `images`
* See images and sizes

### `rmi`
* Remove images
* Ensure no containers are running on that image before deleting it

### `pull`
* To only download an image without running a container of it

### `exec [CMD]`
* Execute commands on running Docker container

### Detached Mode
* `docker run -d`
* Detached mode will run the container in the background
* To attach back to the container: `docker attach [docker container ID]`
* Only need to specify the first few characters of the ID to distinguish it from the other containers

### Port Mapping
* For webapps, Docker will provide an IP and port that can only be accessed inside the Docker host
* To access the app from outside the Docker host, you'll need to map ports

```bash
docker run -p [External port]:[Docker Host port] webapp
```

### Volume Mapping
* Data from app is stored in a folder inside the Docker host
* To ensure data persistence, map to a location outside the Docker host

```bash
docker run -v ExternalDirectory:FolderInDockerContainer webapp
```

### Inspect Container
```bash
docker inspect ContainerName
```

### Check logs
```bash
docker logs ContainerName
```

## Docker Images

### Steps for Creating Image
1. OS - Ubuntu
2. Update apt repo
3. Install dependencies using apt
4. Install Python dependencies using pip
5. Copy source code to /opt folder
6. Run the web server using Flask

### Dockerfile
* Contains instructions for setting up the application
* Must always start from a base OS or another image using a `FROM` instruction

```bash
# Base OS
FROM Ubuntu

# Install dependencies
RUN apt-get update
RUN apt-get install python
RUN pip install flask
RUN pip install flask-mysql

# Copy source code to location in Docker image
COPY . /opt/source-code

# What will be run when the image is run as a container
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

### Building Images
* When Docker builds images using `docker build . -t AccountName/ImageName`, it builds in layers
* Each layer in the Dockerfile is a new layer
* Each layer only stores changes from the previous layer
* Layered architecture allows you to continue from any step so you won't have to start all over again
* If building failed at a certain step, Docker will automatically build earlier successful layers from cache

### Pushing Images

```bash
# Log in first - will prompt for username and password
docker login

# Push to Docker Hub
docker push AccountName/ImageName
```

### Configure Environment Variables
Use the code below to configure environment variables that the app may use:

```bash
docker run -e VAR1=hey AppName
```

### Command vs. Entrypoint

#### Command
Commands are definite. You must specify a command and a parameter. To set the command in the Dockerfile, see the code below. The command line parameters will replace the Dockerfile parameters.

```bash
...
# Option 1
CMD sleep 5

# Option 2
CMD ["sleep", "5"]
```

#### Entrypoint
This will append the command line parameter to the command. If you don't specify a parameter, there will be an error.

```bash
ENTRYPOINT ["sleep"]
```

#### Combine Them
To prevent an error when you don't specify a parameter, use both. To do this, you must specify `ENTRYPOINT` and `CMD` in JSON format.

```bash
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Then, to overwrite the `ENTRYPOINT` command in the command line, use the `--entrypoint` option.

```bash
docker run --entrypoint NewCommand ImageName Parameter
```

## Docker Compose
A file that contains the various images and settings for an application stack to be launched on a single Docker host.

### Compose 
```yaml
# docker-compose.yml
services:
    voting:
        image:  "voting-app"
    results:
        image:  "result-app"
    database:
        image:  "postgres"
    messaging:
        image:  "redis:alpine"
    worker:
        image:  "worker
```

Use the command:

```bash
docker-compose up
```

Doing separate `docker run` calls will not link the containers. We need to link them (e.g. voting app to redis), but this concept is deprecated already. For info:

```bash
# docker run --link [Image]:[Name of Container] AppName
# For voting app container
docker run -d --name=votingapp -p 5000:80 --link redis:redis voting-app

# For results container
docker run -d --name=result-app -p 5001:80 --link db:db result-app

# For worker
docker run -d --name=worker --link db:db --link redis:redis
```

To compose a Docker-compose file from this:

```yaml
redis:
    image: redis
db:
    image: postgres:9.4
vote:
    image: voting-app
    ports:
        - 5000:80
    links:
        - redis
result:
    image: result-app
    ports:
        - 5001:80
    links:
        -db
worker:
    image: worker
    links:
        - redis
        - db
```

Then, run `docker-compose up`.

### Build
Above, we assumed the above were already built. If instead you want to build the `voting-app`, `result-app` and `worker` images before running them as containers, use the `build` key and pass the source containing the source code and instructions.

```yaml
redis:
    image: redis
db:
    image: postgres:9.4
vote:
    build: ./vote
    ports:
        - 5000:80
    links:
        - redis
result:
    build: ./result
    ports:
        - 5001:80
    links:
        -db
worker:
    build: ./worker
    links:
        - redis
        - db
```

### Compose Versions
Must specify the version of the `docker-compose.yml` file at the top.

* Version 2:
    * Must specify `version: [2|3|]` at the top
    * Everything falls under `services`
    * Don't need to use `links` keyword
    * Can use `depends_on` to indicate dependencies
* Version 3:
    * Supports Docker Swarm
    * More details later

```yaml
version: 2
services: 
    redis:
        image: redis
    db:
        image: postgres:9.4
    vote:
        build: ./vote
        ports:
            - 5000:80
        depends_on:
            - redis
    result:
        build: ./result
        ports:
            - 5001:80
    worker:
        build: ./worker
```

### Networks
Suppose `voting-app` and `result-app` are in a front end network, `redis` and `db` are in a backend network.

```yaml
version: 2
services: 
    redis:
        image: redis
        networks:
            - back-end
    db:
        image: postgres:9.4
        networks:
            - back-end
    vote:
        image: voting-app
        ports:
            - 5000:80
        depends_on:
            - redis
        networks:
            - front-end
            - back-end
    result:
        image: result-app
        ports:
            - 5001:80
        networks:
            - front-end
            - back-end
    worker:
        image: worker
        networks:
            - back-end
networks:
    front-end:
    back-end:
```

## Registry
* A central repository of all Docker images
* Without specifying a username, it assumes that it is the same as the image name e.g. `nginx` is actually `nginx/nginx`, where the user/account is `nginx`
* Private registry:
    * Need to login to registry
    * `docker login private-registry.io
* On-prem private registry:
    * Docker Private Registry is available as a Docker image named `registry`
    * Exposes API on port 5000
    * To push image to private registry:

```bash
# Run private registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag image with private registry URL
docker image tag ImageName localhost:5000/ImageName

# Push image
docker push localhost:5000/ImageName

# Pull image
docker pull localhost:5000/ImageName
docker pull 192.168.56.100:5000/ImageName
```

## Docker Engine

### Components
* Docker Daemon
* REST API
* Docker CLI: can connect to a remote docker host that has the API + Daemon

### Containerisation
* Use of namespaces that contain:
    * Process ID (PID)
    * Unix timehsaring
    * Mount
    * Network
    * InterProcess
* PID:
    * Each container has:
        * A PID in the main system
        * Its own set of processes PID 1, 2, etc. inside the container
* Controlling hardware resources:
    * Restricted using `cgroups` (control groups)
    * `--cpus=0.5` means the container must take up no more than 50% of the host's CPU
    * `--memory=100m` means the container must take up no more than 100MB of RAM

### Storage

#### Data on Local File System

* Creates folder `/var/lib/docker` and subfolders for images and containers
* 

#### Layered Architecture
* Each line Dockerfile = new layer in container
* Any images that have the same layers as other images will not be created again - layes will be reused rom cache
* Images are Read Only
* When you run a container, Docker creates a container layer that is read-write
* To modify source code:
    * Docker will create a copy of the source code in the container layer - called Copy-on-write
* If you delete a container, all data inside it will be deleted

#### Persistence
* Create a folder `data_volume` inside the `var/lib/docker/volumes` directory on the Docker host
* When running the container, mount the volume
* If the local folder specified in the command doesn't exist, Docker will create a folder
* Types of mounting:
    * Volume Mounting: A folder from the volume directory
    * Bind Mounting: A folder from any location on the Docker host
* New style is to use `--mount` instead of `-v`
    * `docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql

```bash
# Create volume
docker volume create data_volume

# Map the volume MainHost:DefaultMySQLFolder
docker run -v data_volume:/var/lib/mysql mysql
```

## Networking

### Network Types

#### `Bridge`
* Private internal network inside the container
* Containers can access each other using the IP `172.17.X.X`
* To access the network, either:
    * Map the ports to a port on the Docker host
    * Associate container to the `host` network:
        * No port mapping required
        * Web container uses the host network
        * Cannot run multiple web containers on the same port

##### Custom Networks Within `Bridge`
To create isolated networks within `bridge`

Create subnet:

```bash
docker network create --driver bridge --subnet 182.18.0.0/16 custom-isolated-network-name
```

See all networks:

```bash
docker network ls
```

To see which network each container is attached to:

```bash
docker inspect ContainerID
```

#### `None`
* Containers are not attached to any network
* No access to other containers or external network

### Embedded DNS
* Containers shouldn't refer to each other using IP
* Use the container name

## Docker Swarm

### Problem
* Need to monitor status of containers
* If host crashes, containers are no longer accessible

### Container Orchestration
* Host containers in a production environment
* Deploys many instances of application with a single command
* Scale up instances when users increases, and scale down when users decreases
* Advanced networking between containers across different hosts, load balancing, sharing storage, configuration management, and security
* Techs:
    * Docker Swarm
    * Kubernetes
    * Mesos

```bash
# Launch many instances of an image
docker service create --replicas=100 nodejs
```

### Docker Swarm
* Install Docker Swarm on multiple hosts
* Designate one as Swarm Manager, and the rest will be Workers
* Initialise the manager `docker swarm init`
* Workers should join the swarm (also called nodes now)
* Run on the manager: `docker service create --replicas=3 WebApp

### Kubernetes
* Can scale up and down based on user load
* Can roll forward or roll backward images, and even partition it for A/B testing
* A cluster:
    * Nodes: Virtual/physical node that is a worker machine that launches containers
    * Cluster: Group of nodes groups together; applications can be accessed from other nodes if one node fails
    * Master node: Responsible for orchestration of containers on worker nodes
* Components:
    * API Sever: The frontend for developers
    * etcd Key-Value score: Store data used to manage the cluster e.g. who is node/master
    * Scheduler: Distribute work across nodes
    * Controller: Brain behind orchestration; noticing and responding when endpoint goes down; makes decision to bring up containers
    * Container Runtime: Underlying software used to run containers (typically Docker)
    * kubelet: Agent that runs on each node to ensure containers are running on each node as expected
* Kube Control Tool (`kubectl`): CLI to deploy and manage applications on a Kubernetes cluster