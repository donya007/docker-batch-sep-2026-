# 🐳 Docker Complete Notes & Command Reference

> **Source:** Docker training notes/PDF provided for this repository.  
> This README keeps the concepts and practical commands from the notes and expands them into a structured Docker learning/reference guide.

---

## 📚 Table of Contents

1. [What is Docker?](#1-what-is-docker)
2. [Virtualization vs Containerization](#2-virtualization-vs-containerization)
3. [Why Containerization?](#3-why-containerization)
4. [Docker Architecture](#4-docker-architecture)
5. [Docker Client, Daemon and Registry](#5-docker-client-daemon-and-registry)
6. [Docker Images](#6-docker-images)
7. [Docker Containers](#7-docker-containers)
8. [Dockerfile](#8-dockerfile)
9. [Docker Build](#9-docker-build)
10. [Docker Run](#10-docker-run)
11. [Ports and Port Mapping](#11-ports-and-port-mapping)
12. [Container Lifecycle](#12-container-lifecycle)
13. [Exec and Shell Access](#13-exec-and-shell-access)
14. [Inspecting Docker Objects](#14-inspecting-docker-objects)
15. [Docker Logs](#15-docker-logs)
16. [Copying Files](#16-copying-files)
17. [Docker Images Commands](#17-docker-images-commands)
18. [Container Commands](#18-container-commands)
19. [Docker Networks](#19-docker-networks)
20. [Bridge vs Custom Bridge](#20-bridge-vs-custom-bridge)
21. [Volumes and Persistent Data](#21-volumes-and-persistent-data)
22. [Docker Registry](#22-docker-registry)
23. [Image Save/Load and Export/Import](#23-image-saveload-and-exportimport)
24. [Docker System Commands](#24-docker-system-commands)
25. [Docker Resource Monitoring](#25-docker-resource-monitoring)
26. [Dockerfile Practical Examples](#26-dockerfile-practical-examples)
27. [Common Docker Commands Cheat Sheet](#27-common-docker-commands-cheat-sheet)
28. [Docker Troubleshooting](#28-docker-troubleshooting)
29. [Docker Learning Flow](#29-docker-learning-flow)

---

# 1. What is Docker?

Docker is a **containerization platform**.

The main idea is:

```text
Application
    +
Dependencies
    +
Required runtime
    ↓
Container
```

A container packages what an application needs to run in an isolated environment.

The training notes describe containerization as packaging an application together with its required dependencies so that it can run consistently across systems.

Docker is one containerization technology. Other technologies mentioned in the notes include:

- Docker
- Podman
- containerd
- CRI-O
- Rocket

### Why Docker?

Docker provides a strong CLI and APIs for managing:

- Images
- Containers
- Networks
- Volumes
- Other Docker objects

---

# 2. Virtualization vs Containerization

## Traditional / Bare Metal

**Bare metal** means working directly on physical hardware without an additional virtualization layer.

Example:

```text
Physical Hardware
      ↓
Operating System
      ↓
Application
```

---

## Hardware-Level Virtualization

A hypervisor creates virtual machines.

```text
Physical Hardware
      ↓
Hypervisor
   ↓       ↓
 VM-1     VM-2
  ↓        ↓
Linux    Windows
  ↓        ↓
App      App
```

Each VM normally has its own guest operating system.

Example from the notes:

```text
Hardware
8 GB RAM
8 Core CPU
      ↓
Hypervisor
      ↓
Linux VM       Windows VM
3 GB RAM       3 GB RAM
3 CPU          3 CPU
```

---

## Containerization / OS-Level Virtualization

Containers share the host OS kernel instead of requiring a complete guest OS for every application.

```text
Hardware
   ↓
Host OS
   ↓
Docker Engine
   ↓
Container 1    Container 2
   ↓               ↓
App + deps      App + deps
```

This is generally lighter than running a full VM for every application.

### Core difference

| Virtual Machine | Container |
|---|---|
| Has guest OS | Shares host kernel |
| Heavier | Lightweight |
| More resource overhead | Lower overhead |
| VM isolation | Process/container isolation |
| Boots a complete OS | Starts application process/environment |

---

# 3. Why Containerization?

The notes emphasize **isolation**.

> The whole purpose of containerization is isolation.

A container packages:

```text
Application
+
Dependencies
+
Libraries
+
Runtime
```

This helps reduce "works on my machine" problems by keeping application requirements together.

### Example

Suppose:

```text
App 1 → Java 8
App 2 → Java 17
```

Instead of installing conflicting runtimes on the same OS, containers can package the required runtime/environment separately.

---

# 4. Docker Architecture

High-level Docker architecture:

```text
Developer
    |
    v
Docker Client
    |
    v
Docker Daemon / Docker Engine
    |
    +----------+----------+
    |          |          |
 Images    Containers   Networks
    |
    v
Docker Registry
    |
    v
Docker Hub / Private Registry
```

Important components:

- Docker Client
- Docker Daemon
- Docker Engine
- Images
- Containers
- Registry
- Networks
- Volumes

---

# 5. Docker Client, Daemon and Registry

## Docker Client

The CLI through which we send commands.

Example:

```bash
docker ps
docker images
docker run nginx
```

---

## Docker Daemon

The Docker daemon/engine performs Docker operations.

It manages:

- Containers
- Images
- Networks
- Volumes
- Other Docker resources

---

## Docker Registry

A registry stores Docker images.

Examples:

- Docker Hub
- Private container registries
- Enterprise registries

Example:

```bash
docker pull nginx
```

Docker can download the image from a registry.

---

# 6. Docker Images

An **image** is a packaged template used to create containers.

Think:

```text
Image = Blueprint
Container = Running instance of that blueprint
```

Example:

```text
nginx image
     ↓
container-1
container-2
container-3
```

Multiple containers can be created from the same image.

### Image naming

Example:

```text
nginx:latest
grafana/grafana
ubuntu:22.04
```

General format:

```text
REGISTRY/USERNAME/IMAGE:TAG
```

---

# 7. Docker Containers

A container is a running or created instance based on an image.

Example:

```bash
docker run nginx
```

Flow:

```text
Docker Image
     ↓
docker run
     ↓
Container
```

### Container names

You can assign a name:

```bash
docker run --name mycontainer nginx
```

Without `--name`, Docker can generate a random container name.

---

# 8. Dockerfile

A Dockerfile contains instructions for building an image.

Example:

```dockerfile
FROM tomcat:8.2.20-jre

COPY target/java-web-app*.war /usr/local/tomcat/webapps/java-we-app.war
```

The notes demonstrate this flow:

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
docker run
    ↓
Container
```

### Basic Dockerfile structure

```dockerfile
FROM ubuntu:22.04

RUN apt-get update

COPY app.sh /app/app.sh

CMD ["bash", "/app/app.sh"]
```

---

# 9. Docker Build

Build an image from a Dockerfile:

```bash
docker build -t myimage .
```

Example from the notes:

```bash
docker build -t dockerhandson/java-web-app:1.0 .
```

Meaning:

```text
docker build
    ↓
-t = tag/name
    ↓
dockerhandson/java-web-app:1.0
    ↓
. = current directory/build context
```

### Build with a different Dockerfile

```bash
docker build -f Dockerfile.dev -t myapp:dev .
```

### Disable cache

```bash
docker build --no-cache -t myimage .
```

### Show build help

```bash
docker build --help
```

---

# 10. Docker Run

Basic syntax:

```bash
docker run IMAGE
```

Example:

```bash
docker run nginx
```

Detached mode:

```bash
docker run -d nginx
```

Named container:

```bash
docker run -d --name web nginx
```

Port mapping:

```bash
docker run -d --name web -p 8080:80 nginx
```

Meaning:

```text
Host Port 8080
      |
      v
Container Port 80
```

### Common options

```bash
docker run -d IMAGE
docker run --name NAME IMAGE
docker run -p HOST:CONTAINER IMAGE
docker run -it IMAGE bash
docker run --rm IMAGE
docker run -e KEY=value IMAGE
docker run -v volume:/path IMAGE
```

---

# 11. Ports and Port Mapping

Containers have their own network namespace.

Example:

```bash
docker run -d --name mehak -p 8080:80 nginx
```

Another container:

```bash
docker run -d --name mehak2 -p 8081:80 nginx
```

Result:

```text
Host                 Container
8080  -------------> 80
8081  -------------> 80
```

Both containers can listen on port 80 internally because the containers have separate network namespaces.

### Check port mapping

```bash
docker port mehak
```

```bash
docker ps
```

### Important

`EXPOSE` in a Dockerfile does **not** publish a port to the host.

Publishing is done using:

```bash
-p HOST_PORT:CONTAINER_PORT
```

---

# 12. Container Lifecycle

A useful lifecycle:

```text
Image
  ↓
docker create
  ↓
Created
  ↓
docker start
  ↓
Running
  ↓
docker stop
  ↓
Stopped
  ↓
docker start
  ↓
Running
  ↓
docker rm
  ↓
Removed
```

### Create

```bash
docker create nginx
```

### Start

```bash
docker start CONTAINER
```

### Stop

```bash
docker stop CONTAINER
```

### Restart

```bash
docker restart CONTAINER
```

### Remove

```bash
docker rm CONTAINER
```

Force remove:

```bash
docker rm -f CONTAINER
```

---

# 13. Exec and Shell Access

Run a command inside a running container:

```bash
docker exec CONTAINER ls
```

Example:

```bash
docker exec b3e80865453a ls
```

Check working directory:

```bash
docker exec CONTAINER pwd
```

Interactive shell:

```bash
docker exec -it CONTAINER /bin/bash
```

For images without bash:

```bash
docker exec -it CONTAINER /bin/sh
```

### Difference

```bash
docker exec -it container bash
```

Runs a new process inside the running container.

---

# 14. Inspecting Docker Objects

Inspect a container:

```bash
docker inspect CONTAINER
```

Example:

```bash
docker inspect b2f3e696bd14
```

Inspect an image:

```bash
docker image inspect nginx
```

Inspect a network:

```bash
docker network inspect bridge
```

Inspect a volume:

```bash
docker volume inspect myvolume
```

`inspect` is extremely useful for troubleshooting.

---

# 15. Docker Logs

View logs:

```bash
docker logs CONTAINER
```

Follow logs:

```bash
docker logs -f CONTAINER
```

Show timestamps:

```bash
docker logs -t CONTAINER
```

Show latest lines:

```bash
docker logs --tail 100 CONTAINER
```

---

# 16. Copying Files

Copy from host to container:

```bash
docker cp ./index.html CONTAINER:/usr/share/nginx/html/
```

Copy from container to host:

```bash
docker cp CONTAINER:/path/file.txt .
```

Example:

```bash
docker cp mycontainer:/usr/share/nginx/html/index.html .
```

### COPY vs docker cp

`COPY` is a **Dockerfile instruction** used while building an image.

```dockerfile
COPY index.html /usr/share/nginx/html/
```

`docker cp` is a **CLI command** used to copy files between a running/stopped container and the host.

```bash
docker cp index.html mycontainer:/tmp/
```

---

# 17. Docker Images Commands

List images:

```bash
docker images
```

Modern equivalent:

```bash
docker image ls
```

Pull image:

```bash
docker pull nginx
```

Pull specific tag:

```bash
docker pull ubuntu:22.04
```

Remove image:

```bash
docker rmi IMAGE
```

or:

```bash
docker image rm IMAGE
```

Show image history:

```bash
docker history IMAGE
```

Tag an image:

```bash
docker tag SOURCE_IMAGE TARGET_IMAGE
```

Example:

```bash
docker tag myapp:latest username/myapp:1.0
```

Search Docker Hub:

```bash
docker search nginx
```

Login:

```bash
docker login
```

Logout:

```bash
docker logout
```

Push:

```bash
docker push username/myapp:1.0
```

---

# 18. Container Commands

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

Only container IDs:

```bash
docker ps -q
```

All container IDs:

```bash
docker ps -aq
```

Start:

```bash
docker start CONTAINER
```

Stop:

```bash
docker stop CONTAINER
```

Restart:

```bash
docker restart CONTAINER
```

Remove:

```bash
docker rm CONTAINER
```

Force remove:

```bash
docker rm -f CONTAINER
```

View processes:

```bash
docker top CONTAINER
```

Pause:

```bash
docker pause CONTAINER
```

Unpause:

```bash
docker unpause CONTAINER
```

Kill:

```bash
docker kill CONTAINER
```

Rename:

```bash
docker rename OLD_NAME NEW_NAME
```

Wait for container exit:

```bash
docker wait CONTAINER
```

---

# 19. Docker Networks

Docker provides networking for container communication.

List networks:

```bash
docker network ls
```

The notes demonstrate:

```bash
docker network create
docker network connect
docker network disconnect
docker network ls
docker network inspect
docker network prune
docker network rm
```

### Create a network

```bash
docker network create mentor-network
```

### Create custom bridge network

```bash
docker network create --driver bridge my-network
```

### Connect a container

```bash
docker network connect my-network container1
```

### Disconnect

```bash
docker network disconnect my-network container1
```

### Inspect

```bash
docker network inspect my-network
```

### Remove

```bash
docker network rm my-network
```

### Remove unused networks

```bash
docker network prune
```

---

# 20. Bridge vs Custom Bridge

## Default bridge

Docker commonly has a default `bridge` network.

Example:

```bash
docker network ls
```

Typical networks include:

```text
bridge
host
none
```

The notes show containers such as:

```text
c1
c2
```

connected through Docker networking.

---

## Custom Bridge

Create:

```bash
docker network create mentor-network
```

Run containers on it:

```bash
docker run -d --name frontend --network mentor-network nginx
docker run -d --name backend --network mentor-network nginx
```

Now containers on the same custom network can communicate using container names.

Example:

```bash
curl http://backend
```

This is one major advantage of user-defined bridge networks.

### IP communication

The notes also demonstrate communication using container IP addresses, for example:

```bash
curl -v 172.17.0.2:5000/
```

and name-based communication:

```bash
curl -v netflix1:80
```

---

# 21. Volumes and Persistent Data

Containers are generally treated as disposable.

For persistent data, use volumes.

Create volume:

```bash
docker volume create myvolume
```

List volumes:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect myvolume
```

Run container with volume:

```bash
docker run -d \
  --name mysql \
  -v myvolume:/var/lib/mysql \
  mysql
```

Remove volume:

```bash
docker volume rm myvolume
```

Remove unused volumes:

```bash
docker volume prune
```

### Bind mount

```bash
docker run -d \
  -v /host/path:/container/path \
  nginx
```

### Volume vs bind mount

```text
Named Volume
Docker manages storage

Bind Mount
You specify host filesystem path
```

---

# 22. Docker Registry

A registry stores Docker images.

Docker Hub is a common public registry.

Example:

```bash
docker pull nginx
```

Build:

```bash
docker build -t username/myapp:1.0 .
```

Login:

```bash
docker login
```

Push:

```bash
docker push username/myapp:1.0
```

Pull:

```bash
docker pull username/myapp:1.0
```

Flow:

```text
Developer
   ↓
docker build
   ↓
Image
   ↓
docker tag
   ↓
Registry
   ↓
docker push
   ↓
Other system
   ↓
docker pull
```

---

# 23. Image Save/Load and Export/Import

These commands are commonly confused.

## docker save

Saves an image as a tar archive.

```bash
docker save -o myimage.tar myimage:latest
```

Load it:

```bash
docker load -i myimage.tar
```

Use this for moving Docker **images**.

---

## docker export

Exports a container filesystem.

```bash
docker export CONTAINER -o container.tar
```

Import:

```bash
docker import container.tar myimage:latest
```

### Difference

```text
docker save
    ↓
IMAGE → TAR

docker load
    ↓
TAR → IMAGE


docker export
    ↓
CONTAINER → TAR

docker import
    ↓
TAR → IMAGE
```

---

# 24. Docker System Commands

Show Docker information:

```bash
docker info
```

Docker version:

```bash
docker version
```

Docker help:

```bash
docker --help
```

System disk usage:

```bash
docker system df
```

Remove unused resources:

```bash
docker system prune
```

More aggressive cleanup:

```bash
docker system prune -a
```

Be careful: cleanup commands can remove resources you still need.

---

# 25. Docker Resource Monitoring

Live resource usage:

```bash
docker stats
```

Specific container:

```bash
docker stats CONTAINER
```

Show processes:

```bash
docker top CONTAINER
```

Useful for checking:

- CPU
- Memory
- Network I/O
- Block I/O
- Process count

---

# 26. Dockerfile Practical Examples

## Example 1 — Nginx

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Build:

```bash
docker build -t mynginx:1.0 .
```

Run:

```bash
docker run -d --name mynginx -p 8080:80 mynginx:1.0
```

---

## Example 2 — Java/Tomcat WAR Application

Based on the training notes:

```dockerfile
FROM tomcat:8.2.20-jre

COPY target/java-web-app*.war /usr/local/tomcat/webapps/java-we-app.war
```

Build:

```bash
docker build -t dockerhandson/java-web-app:1.0 .
```

Run:

```bash
docker run -d \
  --name javaapp \
  -p 8080:8080 \
  dockerhandson/java-web-app:1.0
```

---

## Example 3 — Ubuntu

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y nginx

CMD ["nginx", "-g", "daemon off;"]
```

Build:

```bash
docker build -t ubuntu-nginx:1.0 .
```

---

# 27. Common Docker Commands Cheat Sheet

## Information

```bash
# Show Docker version
docker version

# Show Docker system information
docker info

# Show Docker CLI help
docker --help

# Help for a specific command
docker run --help
```

## Images

```bash
# List images
docker images

# Pull an image
docker pull nginx

# Remove an image
docker rmi nginx

# Show image history
docker history nginx

# Inspect an image
docker image inspect nginx

# Tag an image
docker tag myapp:latest username/myapp:1.0

# Push an image
docker push username/myapp:1.0
```

## Containers

```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Show only IDs of running containers
docker ps -q

# Show IDs of all containers
docker ps -aq

# Run a container
docker run nginx

# Run in background
docker run -d nginx

# Create named container
docker run -d --name web nginx

# Stop container
docker stop web

# Start container
docker start web

# Restart container
docker restart web

# Remove container
docker rm web

# Force remove container
docker rm -f web
```

## Shell / Execution

```bash
# Execute a command
docker exec web ls

# Open Bash shell
docker exec -it web /bin/bash

# Open sh shell
docker exec -it web /bin/sh
```

## Logs

```bash
# Show logs
docker logs web

# Follow logs
docker logs -f web

# Show last 100 lines
docker logs --tail 100 web
```

## Inspect

```bash
docker inspect web
docker image inspect nginx
docker network inspect bridge
docker volume inspect myvolume
```

## Ports

```bash
# Publish host 8080 to container 80
docker run -d -p 8080:80 nginx

# Show container ports
docker port web
```

## Copy

```bash
docker cp index.html web:/usr/share/nginx/html/
docker cp web:/usr/share/nginx/html/index.html .
```

## Network

```bash
docker network ls
docker network create my-network
docker network connect my-network web
docker network disconnect my-network web
docker network inspect my-network
docker network rm my-network
docker network prune
```

## Volume

```bash
docker volume ls
docker volume create myvolume
docker volume inspect myvolume
docker volume rm myvolume
docker volume prune
```

## Cleanup

```bash
docker system df
docker system prune
docker system prune -a
docker container prune
docker image prune
docker network prune
docker volume prune
```

---

# 28. Docker Troubleshooting

## Check whether Docker is working

```bash
docker version
docker info
```

## Test Docker

```bash
docker run hello-world
```

## Check containers

```bash
docker ps
docker ps -a
```

## Check images

```bash
docker images
```

## Check logs

```bash
docker logs CONTAINER
```

## Inspect container

```bash
docker inspect CONTAINER
```

## Check port mapping

```bash
docker port CONTAINER
docker ps
```

## Check network

```bash
docker network ls
docker network inspect NETWORK
```

## Enter container

```bash
docker exec -it CONTAINER /bin/bash
```

If bash does not exist:

```bash
docker exec -it CONTAINER /bin/sh
```

## Find Docker executable

Linux:

```bash
which docker
```

The training notes also demonstrate:

```bash
which nginx
```

---

# 29. Docker Learning Flow

Recommended order for hands-on practice:

```text
1. Docker installation
        ↓
2. docker version
        ↓
3. docker info
        ↓
4. docker run hello-world
        ↓
5. Images
        ↓
6. Containers
        ↓
7. docker ps / docker ps -a
        ↓
8. docker exec
        ↓
9. docker logs
        ↓
10. docker inspect
        ↓
11. Port mapping
        ↓
12. Dockerfile
        ↓
13. docker build
        ↓
14. Docker networks
        ↓
15. Docker volumes
        ↓
16. Docker registry
        ↓
17. Image push/pull
        ↓
18. Docker Compose
        ↓
19. Multi-stage builds
        ↓
20. Docker security
        ↓
21. Docker + CI/CD
        ↓
22. Docker + Kubernetes
```

---

# 🎯 Practical Mini Lab

## Step 1 — Pull Nginx

```bash
docker pull nginx
```

## Step 2 — Run Nginx

```bash
docker run -d --name nginx1 -p 8080:80 nginx
```

## Step 3 — Check

```bash
docker ps
```

## Step 4 — Logs

```bash
docker logs nginx1
```

## Step 5 — Enter container

```bash
docker exec -it nginx1 /bin/bash
```

## Step 6 — Check files

```bash
ls
cd /usr/share/nginx/html
ls
```

## Step 7 — Exit

```bash
exit
```

## Step 8 — Stop

```bash
docker stop nginx1
```

## Step 9 — Start again

```bash
docker start nginx1
```

## Step 10 — Remove

```bash
docker rm -f nginx1
```

---

# 🧠 Important Docker Concepts in One Diagram

```text
                 Docker Registry
                       |
                  docker pull
                       |
                       v
                 Docker Image
                       |
                 docker run
                       |
                       v
                  Container
                 /    |     \
                /     |      \
           Network   Volume   Port
              |        |        |
              v        v        v
          Container  Data    Host:Port
          Communication
```

---

# 🔥 Most Important Interview Differences

## Image vs Container

```text
Image     → Template / blueprint
Container → Running/created instance of image
```

## VM vs Container

```text
VM         → Full guest OS
Container  → Shares host kernel
```

## docker run vs docker start

```text
docker run
→ Creates + starts a new container

docker start
→ Starts an existing stopped container
```

## docker stop vs docker kill

```text
docker stop
→ Graceful stop

docker kill
→ Immediate termination signal
```

## docker rm vs docker rmi

```text
docker rm
→ Removes container

docker rmi
→ Removes image
```

## docker exec vs docker attach

```text
docker exec
→ Starts a new process inside container

docker attach
→ Attaches to the container's existing main process
```

## docker save vs docker export

```text
docker save
→ Image → tar

docker export
→ Container filesystem → tar
```

## COPY vs docker cp

```text
COPY
→ Dockerfile instruction during image build

docker cp
→ CLI command to copy files between host and container
```

---

# 📌 Docker CLI Command Categories

From `docker --help`, important command groups include:

```text
docker builder
docker buildx
docker container
docker context
docker image
docker manifest
docker network
docker plugin
docker system
docker volume
docker swarm
```

Common standalone commands include:

```text
run
exec
ps
build
pull
push
images
login
logout
search
version
info
attach
commit
cp
create
diff
events
export
history
import
inspect
kill
load
logs
pause
port
rename
restart
rm
rmi
save
start
stats
stop
tag
top
unpause
update
wait
```

---

# 🚀 Quick Reference

If you remember only these commands first:

```bash
docker version
docker info

docker pull nginx
docker images

docker run -d --name web -p 8080:80 nginx

docker ps
docker ps -a

docker exec -it web /bin/bash
docker logs web
docker inspect web
docker port web

docker stop web
docker start web
docker restart web
docker rm web

docker build -t myimage:1.0 .
docker rmi myimage:1.0

docker network ls
docker network create my-network
docker network connect my-network web
docker network inspect my-network

docker volume ls
docker volume create myvolume
docker volume inspect myvolume

docker login
docker tag myimage:1.0 username/myimage:1.0
docker push username/myimage:1.0
docker pull username/myimage:1.0
```

---

# 📖 Source Notes

This README is based on the uploaded Docker training PDF. The source covers:

- Bare metal and virtualization
- Hypervisor and guest OS concepts
- Containerization and isolation
- Docker architecture
- Dockerfile → Image → Container flow
- Docker Client, Daemon and Registry
- Port mapping
- Core Docker commands
- Container lifecycle
- Docker `exec`, `inspect`, `logs`, `cp`
- Docker images
- Docker networks
- Default/custom bridge networking
- Container-to-container communication
- Docker command categories

The source specifically demonstrates commands such as `docker build`, `docker run`, `docker ps`, `docker exec`, `docker inspect`, `docker network create/connect/disconnect/inspect`, and image/container management commands. fileciteturn0file0L114-L124 fileciteturn0file0L185-L233 fileciteturn0file0L235-L294 fileciteturn0file0L336-L386

---

## ⭐ Next-Level Docker Topics

For a complete production/DevOps Docker curriculum, continue with:

- Docker Compose
- Multi-stage Docker builds
- BuildKit / Buildx
- Dockerfile optimization
- Image layers and caching
- `.dockerignore`
- Environment variables
- Build arguments
- CMD vs ENTRYPOINT
- Shell form vs Exec form
- HEALTHCHECK
- USER and non-root containers
- Resource limits
- Docker security
- Docker secrets
- Container logging drivers
- Private registries
- CI/CD with Docker
- Docker image scanning
- Docker + Azure
- Docker + Kubernetes
- Production container best practices
