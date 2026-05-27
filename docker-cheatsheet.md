# 🐳 Docker Commands: Basic to Advanced

A comprehensive reference guide covering almost everything you need in Docker.

---

## Table of Contents

1. [Command Quick Reference Table](#1-command-quick-reference-table)
2. [Installing Docker](#2-installing-docker)
3. [Docker CLI Basics](#3-docker-cli-basics)
4. [Images](#4-images)
5. [Containers](#5-containers)
6. [Volumes](#6-volumes)
7. [Networks](#7-networks)
8. [Dockerfile](#8-dockerfile)
9. [Docker Compose](#9-docker-compose)
10. [Registry & Image Sharing](#10-registry--image-sharing)
11. [Docker Buildx & Multi-Platform](#11-docker-buildx--multi-platform)
12. [Resource Management & Limits](#12-resource-management--limits)
13. [Logging & Monitoring](#13-logging--monitoring)
14. [Security](#14-security)
15. [Docker Contexts](#15-docker-contexts)
16. [Docker Swarm](#16-docker-swarm)
17. [Pruning & Cleanup](#17-pruning--cleanup)
18. [Performance & Debugging](#18-performance--debugging)
19. [Advanced Features](#19-advanced-features)

---


## 1. Command Quick Reference Table

### 🖼️ Image Commands

| Command | Use |
|---------|-----|
| `docker images` | List all local images |
| `docker pull <image>` | Download an image from a registry |
| `docker push <image>` | Upload an image to a registry |
| `docker build -t <name> .` | Build an image from a Dockerfile |
| `docker rmi <image>` | Remove an image |
| `docker tag <image> <new_tag>` | Tag an image with a new name |
| `docker save -o file.tar <image>` | Export an image to a tar file |
| `docker load -i file.tar` | Import an image from a tar file |
| `docker inspect <image>` | Show detailed image metadata |
| `docker history <image>` | Show image layer history |
| `docker image prune` | Remove unused (dangling) images |
| `docker image prune -a` | Remove all unused images |
| `docker search <term>` | Search Docker Hub for images |

---

### 📦 Container Commands

| Command | Use |
|---------|-----|
| `docker run <image>` | Create and start a new container |
| `docker run -d <image>` | Run container in background (detached) |
| `docker run -it <image> bash` | Run container with interactive terminal |
| `docker run --name <name> <image>` | Run container with a custom name |
| `docker run -p 8080:80 <image>` | Map host port 8080 to container port 80 |
| `docker run -v /host:/container <image>` | Mount a host directory as a volume |
| `docker run --rm <image>` | Auto-remove container when it stops |
| `docker run -e VAR=value <image>` | Set an environment variable |
| `docker run --env-file .env <image>` | Load env vars from a file |
| `docker run --network <net> <image>` | Connect container to a network |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker start <container>` | Start a stopped container |
| `docker stop <container>` | Gracefully stop a running container |
| `docker restart <container>` | Restart a container |
| `docker kill <container>` | Forcefully stop a container (SIGKILL) |
| `docker rm <container>` | Remove a stopped container |
| `docker rm -f <container>` | Force remove a running container |
| `docker exec -it <container> bash` | Open shell in running container |
| `docker exec <container> <cmd>` | Run a command in a running container |
| `docker logs <container>` | View container logs |
| `docker logs -f <container>` | Stream/follow container logs |
| `docker logs --tail 50 <container>` | View last 50 lines of logs |
| `docker inspect <container>` | Show detailed container metadata |
| `docker stats` | Live resource usage of all containers |
| `docker stats <container>` | Live resource usage of one container |
| `docker top <container>` | Show running processes inside container |
| `docker diff <container>` | Show filesystem changes in container |
| `docker cp <container>:/path ./local` | Copy file from container to host |
| `docker cp ./local <container>:/path` | Copy file from host to container |
| `docker commit <container> <image>` | Create image from container state |
| `docker rename <old> <new>` | Rename a container |
| `docker pause <container>` | Pause all processes in a container |
| `docker unpause <container>` | Unpause a paused container |
| `docker wait <container>` | Block until container stops; print exit code |
| `docker port <container>` | Show port mappings of a container |
| `docker attach <container>` | Attach terminal to running container |
| `docker update <container>` | Update container resource limits |
| `docker container prune` | Remove all stopped containers |

---

### 💾 Volume Commands

| Command | Use |
|---------|-----|
| `docker volume create <name>` | Create a named volume |
| `docker volume ls` | List all volumes |
| `docker volume inspect <name>` | Show volume metadata |
| `docker volume rm <name>` | Remove a volume |
| `docker volume prune` | Remove all unused volumes |
| `docker run -v myvolume:/path <image>` | Mount a named volume |
| `docker run -v /host:/container <image>` | Bind mount a host directory |
| `docker run --mount type=bind,...` | Explicit mount syntax |
| `docker run --tmpfs /tmp <image>` | Mount a temporary in-memory filesystem |

---

### 🌐 Network Commands

| Command | Use |
|---------|-----|
| `docker network create <name>` | Create a custom network |
| `docker network ls` | List all networks |
| `docker network inspect <name>` | Show network details |
| `docker network rm <name>` | Remove a network |
| `docker network prune` | Remove all unused networks |
| `docker network connect <net> <container>` | Connect container to a network |
| `docker network disconnect <net> <container>` | Disconnect container from network |
| `docker run --network host <image>` | Use host's network stack directly |
| `docker run --network none <image>` | Run container with no network |

---

### 🐙 Docker Compose Commands

| Command | Use |
|---------|-----|
| `docker compose up` | Create and start all services |
| `docker compose up -d` | Start all services in background |
| `docker compose up --build` | Rebuild images then start |
| `docker compose down` | Stop and remove containers and networks |
| `docker compose down -v` | Also remove named volumes |
| `docker compose down --rmi all` | Also remove built images |
| `docker compose start` | Start existing stopped services |
| `docker compose stop` | Stop running services (keep containers) |
| `docker compose restart` | Restart all services |
| `docker compose ps` | List services and their status |
| `docker compose logs` | View logs from all services |
| `docker compose logs -f <service>` | Follow logs of a specific service |
| `docker compose exec <service> bash` | Open shell in a running service |
| `docker compose run <service> <cmd>` | Run one-off command in a service |
| `docker compose build` | Build or rebuild images |
| `docker compose pull` | Pull all service images |
| `docker compose push` | Push all service images |
| `docker compose config` | Validate and view composed config |
| `docker compose scale <svc>=N` | Scale a service to N replicas |
| `docker compose top` | Show running processes per service |
| `docker compose pause` | Pause all services |
| `docker compose unpause` | Unpause all services |
| `docker compose rm` | Remove stopped service containers |
| `docker compose events` | Stream real-time events from services |
| `docker compose port <svc> <port>` | Show public port for a service |
| `docker compose version` | Show Docker Compose version |

---

### 🏭 Registry Commands

| Command | Use |
|---------|-----|
| `docker login` | Log in to Docker Hub |
| `docker login <registry>` | Log in to a custom registry |
| `docker logout` | Log out from registry |
| `docker pull <image>:<tag>` | Pull a specific image tag |
| `docker push <image>:<tag>` | Push image to registry |
| `docker search <term>` | Search Docker Hub |
| `docker manifest inspect <image>` | Inspect multi-arch manifest |

---

### 🔧 System Commands

| Command | Use |
|---------|-----|
| `docker version` | Show Docker client and server version |
| `docker info` | Show system-wide Docker info |
| `docker system df` | Show Docker disk usage |
| `docker system prune` | Remove all unused Docker objects |
| `docker system prune -a` | Remove all unused objects including images |
| `docker system prune --volumes` | Also prune volumes |
| `docker system events` | Stream real-time Docker events |
| `docker context ls` | List Docker contexts |
| `docker context use <name>` | Switch Docker context |
| `docker buildx ls` | List builders |
| `docker buildx build --platform` | Build for multiple platforms |
| `docker scan <image>` | Scan image for vulnerabilities |
| `docker sbom <image>` | Generate software bill of materials |

---

### 🐝 Docker Swarm Commands

| Command | Use |
|---------|-----|
| `docker swarm init` | Initialize a Swarm (become manager) |
| `docker swarm join --token <token> <host>` | Join a Swarm as worker |
| `docker swarm leave` | Leave the Swarm |
| `docker swarm leave --force` | Force-leave (manager node) |
| `docker service create <image>` | Create a new service |
| `docker service ls` | List all services |
| `docker service ps <service>` | List tasks of a service |
| `docker service inspect <service>` | Show service details |
| `docker service update <service>` | Update a service |
| `docker service scale <svc>=N` | Scale a service to N replicas |
| `docker service rm <service>` | Remove a service |
| `docker service logs <service>` | View service logs |
| `docker node ls` | List Swarm nodes |
| `docker node inspect <node>` | Show node details |
| `docker node update <node>` | Update node role or availability |
| `docker node rm <node>` | Remove a node from Swarm |
| `docker stack deploy -c file.yml <name>` | Deploy a stack from compose file |
| `docker stack ls` | List stacks |
| `docker stack ps <stack>` | List tasks in a stack |
| `docker stack services <stack>` | List services in a stack |
| `docker stack rm <stack>` | Remove a stack |
| `docker secret create <name> <file>` | Create a secret |
| `docker secret ls` | List secrets |
| `docker config create <name> <file>` | Create a config |
| `docker config ls` | List configs |

---

## 2. Installing Docker

```bash
# Ubuntu / Debian
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add current user to docker group (no sudo needed)
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker version
docker run hello-world

# CentOS / RHEL
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker

# macOS — Download Docker Desktop from https://docs.docker.com/desktop/mac/install/

# Windows — Download Docker Desktop from https://docs.docker.com/desktop/windows/install/

# Check Docker service status
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl stop docker
sudo systemctl restart docker
sudo systemctl enable docker   # Start on boot
```

---

## 3. Docker CLI Basics

```bash
# Docker version info
docker version
docker --version
docker info

# Get help
docker help
docker <command> --help
docker run --help

# Docker disk usage
docker system df
docker system df -v    # Verbose

# System events (real-time stream)
docker system events
docker system events --filter type=container
docker system events --since "2024-01-01" --until "2024-12-31"

# Docker config location
cat ~/.docker/config.json

# Environment variables Docker respects
DOCKER_HOST           # Override Docker daemon socket
DOCKER_TLS_VERIFY     # Enable TLS
DOCKER_CERT_PATH      # Path to TLS certs
DOCKER_BUILDKIT=1     # Enable BuildKit
COMPOSE_FILE          # Default compose file path
```

---

## 4. Images

```bash
# List images
docker images
docker image ls
docker images -a              # Include intermediate images
docker images --digests       # Show image digests
docker images --filter dangling=true   # Only dangling images

# Pull image
docker pull ubuntu
docker pull ubuntu:22.04
docker pull ubuntu:latest
docker pull nginx:1.25-alpine
docker pull myregistry.com:5000/myimage:tag

# Search Docker Hub
docker search nginx
docker search --filter stars=100 nginx
docker search --filter is-official=true python

# Build image from Dockerfile
docker build .
docker build -t myapp:1.0 .
docker build -t myapp:latest -f Dockerfile.prod .
docker build --no-cache -t myapp .
docker build --build-arg ENV=production -t myapp .
docker build --target builder -t myapp-builder .    # Multi-stage target

# Tag an image
docker tag myapp:latest myapp:1.0
docker tag myapp myregistry.com/myapp:latest

# Remove image
docker rmi nginx
docker rmi nginx:1.25
docker rmi $(docker images -q)          # Remove all images
docker rmi $(docker images -f "dangling=true" -q)  # Remove dangling

# Inspect image
docker inspect nginx
docker inspect --format='{{.Os}}' nginx
docker inspect --format='{{json .Config.Env}}' nginx

# Image history (layers)
docker history nginx
docker history --no-trunc nginx

# Save image to tar
docker save -o nginx.tar nginx:latest
docker save nginx:latest | gzip > nginx.tar.gz

# Load image from tar
docker load -i nginx.tar
docker load < nginx.tar.gz

# Export container filesystem as tar (different from save)
docker export mycontainer -o mycontainer.tar

# Import as image from tar
docker import mycontainer.tar myimage:latest

# Image digest
docker images --digests nginx
docker pull nginx@sha256:abc123...

# Flatten image (reduce layers)
docker export mycontainer | docker import - myflattened:latest
```

---

## 5. Containers

### Running Containers

```bash
# Basic run
docker run nginx
docker run ubuntu echo "Hello World"

# Detached mode (background)
docker run -d nginx

# Interactive + TTY
docker run -it ubuntu bash
docker run -it ubuntu /bin/sh

# Named container
docker run --name my-nginx -d nginx

# Auto-remove after exit
docker run --rm ubuntu echo "temporary"

# Port mapping
docker run -p 8080:80 nginx             # host:container
docker run -p 127.0.0.1:8080:80 nginx  # Bind to localhost only
docker run -P nginx                     # Auto-map all exposed ports

# Environment variables
docker run -e MYSQL_ROOT_PASSWORD=secret mysql
docker run -e DB_HOST=localhost -e DB_PORT=5432 myapp
docker run --env-file .env myapp

# Volume mounts
docker run -v /host/data:/container/data nginx
docker run -v myvolume:/app/data nginx
docker run --mount type=bind,source=/host,target=/container nginx
docker run --mount type=volume,source=myvolume,target=/data nginx
docker run --tmpfs /tmp nginx

# Resource limits
docker run --memory="512m" --cpus="1.5" nginx
docker run --memory="1g" --memory-swap="2g" nginx

# Restart policy
docker run --restart always nginx
docker run --restart unless-stopped nginx
docker run --restart on-failure:3 nginx

# Working directory
docker run -w /app myapp

# User
docker run -u 1000:1000 myapp
docker run -u nobody myapp

# Labels
docker run --label app=backend --label env=prod nginx

# Hostname and DNS
docker run --hostname mycontainer nginx
docker run --dns 8.8.8.8 nginx
docker run --add-host myhost:192.168.1.10 nginx

# Capabilities
docker run --cap-add SYS_PTRACE myapp
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp

# Privileged (full host access — use with caution)
docker run --privileged myapp

# Read-only filesystem
docker run --read-only --tmpfs /tmp myapp

# Network
docker run --network mynetwork nginx
docker run --network host nginx
docker run --network none nginx

# SHM size (shared memory)
docker run --shm-size=256m myapp

# Entrypoint override
docker run --entrypoint /bin/sh nginx

# Logging driver
docker run --log-driver json-file --log-opt max-size=10m nginx
```

### Managing Containers

```bash
# List containers
docker ps                     # Running only
docker ps -a                  # All containers
docker ps -q                  # Only IDs
docker ps --filter status=exited
docker ps --filter name=nginx
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Start / stop / restart
docker start mycontainer
docker stop mycontainer                  # Sends SIGTERM, waits 10s, then SIGKILL
docker stop -t 30 mycontainer           # Custom timeout
docker restart mycontainer
docker restart --time 5 mycontainer

# Kill (immediate)
docker kill mycontainer                  # Sends SIGKILL
docker kill --signal SIGTERM mycontainer

# Pause / unpause
docker pause mycontainer
docker unpause mycontainer

# Remove containers
docker rm mycontainer
docker rm -f mycontainer                # Force remove running
docker rm $(docker ps -aq)             # Remove all stopped
docker rm $(docker ps -aq -f status=exited)

# Execute commands in running container
docker exec mycontainer ls /app
docker exec -it mycontainer bash
docker exec -it mycontainer sh
docker exec -u root -it mycontainer bash
docker exec -e DEBUG=true mycontainer ./run.sh
docker exec -w /app mycontainer pwd

# Attach to container (stdin/stdout of PID 1)
docker attach mycontainer
# Detach without stopping: Ctrl+P then Ctrl+Q

# Rename container
docker rename oldname newname

# Copy files
docker cp mycontainer:/app/config.json ./config.json
docker cp ./config.json mycontainer:/app/config.json

# Inspect container
docker inspect mycontainer
docker inspect --format='{{.NetworkSettings.IPAddress}}' mycontainer
docker inspect --format='{{json .Mounts}}' mycontainer

# Container logs
docker logs mycontainer
docker logs -f mycontainer               # Follow
docker logs --tail 100 mycontainer       # Last 100 lines
docker logs --since "2024-01-01" mycontainer
docker logs --since 1h mycontainer       # Last 1 hour
docker logs -t mycontainer               # Show timestamps

# Stats (live resource usage)
docker stats
docker stats mycontainer
docker stats --no-stream                 # One-time snapshot
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Processes inside container
docker top mycontainer
docker top mycontainer aux

# Changes to filesystem
docker diff mycontainer   # A=added, C=changed, D=deleted

# Wait for container to stop
docker wait mycontainer

# Port mappings
docker port mycontainer
docker port mycontainer 80

# Update container resources (live)
docker update --memory 1g mycontainer
docker update --cpus 2 mycontainer
docker update --restart always mycontainer

# Commit container as image
docker commit mycontainer myimage:v2
docker commit -m "Added config" -a "Author" mycontainer myimage:v2

# Export container state
docker export mycontainer > mycontainer.tar
```

---

## 6. Volumes

```bash
# Create volume
docker volume create myvolume
docker volume create --driver local myvolume
docker volume create --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/share \
  nfs-volume

# List volumes
docker volume ls
docker volume ls --filter dangling=true

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume
docker volume rm $(docker volume ls -q)   # Remove all

# Prune unused volumes
docker volume prune
docker volume prune --force

# Named volume (persists across container restarts)
docker run -v myvolume:/app/data myapp

# Anonymous volume (removed with --rm)
docker run -v /app/data myapp

# Bind mount (host directory)
docker run -v $(pwd):/app myapp
docker run -v /absolute/path:/container/path myapp

# Read-only mount
docker run -v myvolume:/app/data:ro myapp

# Long syntax (--mount)
docker run --mount type=volume,source=myvolume,target=/app/data myapp
docker run --mount type=bind,source=$(pwd),target=/app myapp
docker run --mount type=tmpfs,target=/tmp,tmpfs-size=100m myapp

# Share volume between containers
docker run -d --name db -v dbdata:/var/lib/postgresql/data postgres
docker run -d --name db-backup --volumes-from db ubuntu

# Backup a volume
docker run --rm -v myvolume:/data -v $(pwd):/backup ubuntu \
  tar czf /backup/myvolume_backup.tar.gz -C /data .

# Restore a volume
docker run --rm -v myvolume:/data -v $(pwd):/backup ubuntu \
  tar xzf /backup/myvolume_backup.tar.gz -C /data
```

---

## 7. Networks

```bash
# List networks
docker network ls

# Inspect network
docker network inspect bridge
docker network inspect mynetwork

# Create network
docker network create mynetwork
docker network create --driver bridge mynetwork
docker network create --driver overlay mynetwork     # Swarm only
docker network create --subnet 192.168.1.0/24 mynetwork
docker network create \
  --subnet 172.20.0.0/16 \
  --ip-range 172.20.240.0/20 \
  --gateway 172.20.0.1 \
  mynetwork

# Remove network
docker network rm mynetwork
docker network prune        # Remove all unused

# Connect container to network
docker network connect mynetwork mycontainer
docker network connect --ip 172.20.0.10 mynetwork mycontainer

# Disconnect container from network
docker network disconnect mynetwork mycontainer

# Run with specific network
docker run --network mynetwork myapp
docker run --network host myapp       # Use host network
docker run --network none myapp       # No network

# Container DNS (containers resolve each other by name on custom networks)
docker network create mynet
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp
# app can reach db at hostname "db"

# Network drivers
# bridge  — Default. Isolated network on single host
# host    — Shares host network namespace (no isolation)
# overlay — Multi-host networking (Swarm)
# macvlan — Assign MAC address, appear as physical device
# none    — No network access
# ipvlan  — Like macvlan but shares MAC address

# MacVLAN network
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macvlan_net
```

---

## 8. Dockerfile

```dockerfile
# ─────────────────────────────────────────────────
# Basic Dockerfile structure
# ─────────────────────────────────────────────────

# Base image
FROM ubuntu:22.04

# Metadata
LABEL maintainer="you@example.com"
LABEL version="1.0"
LABEL description="My application"

# Set environment variables
ENV NODE_ENV=production \
    APP_PORT=3000 \
    APP_HOME=/app

# Set working directory
WORKDIR /app

# Build arguments (only during build)
ARG NODE_VERSION=18
ARG BUILD_DATE

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Copy files
COPY . .
COPY package.json package-lock.json ./
COPY --chown=node:node . .

# ADD (like COPY but can extract tar and fetch URLs)
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /app/

# Run commands
RUN npm install
RUN npm run build

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# Expose port (documentation only — doesn't publish)
EXPOSE 3000
EXPOSE 3000/tcp
EXPOSE 53/udp

# Mount point for volumes
VOLUME ["/app/data"]

# Healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Disable healthcheck
HEALTHCHECK NONE

# Default command (overridable)
CMD ["node", "server.js"]
CMD ["npm", "start"]

# Entry point (not overridable without --entrypoint)
ENTRYPOINT ["node"]
CMD ["server.js"]    # Default argument to ENTRYPOINT

# Shell form (runs in /bin/sh -c)
CMD node server.js
RUN npm install

# Exec form (direct, no shell — preferred)
CMD ["node", "server.js"]
RUN ["npm", "install"]

# ONBUILD — triggers when image is used as base
ONBUILD COPY . /app
ONBUILD RUN npm install

# STOPSIGNAL — signal to stop container
STOPSIGNAL SIGTERM

# Shell — change default shell
SHELL ["/bin/bash", "-c"]
```

### Multi-Stage Build

```dockerfile
# ─────────────────────────────────────────────────
# Multi-Stage Build — smaller production image
# ─────────────────────────────────────────────────

# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Test
FROM builder AS tester
RUN npm test

# Stage 3: Production (only artifacts, no dev tools)
FROM node:18-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json .
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

```bash
# Build specific stage
docker build --target builder -t myapp-builder .
docker build --target production -t myapp:prod .
```

### .dockerignore

```
# .dockerignore — exclude from build context
node_modules
npm-debug.log
.git
.gitignore
.env
.env.*
*.md
Dockerfile
.dockerignore
dist
coverage
.nyc_output
*.test.js
**/__tests__
.DS_Store
```

### Dockerfile Best Practices

```dockerfile
# ✅ Combine RUN commands to reduce layers
RUN apt-get update && apt-get install -y \
    curl \
    git \
    vim \
    && rm -rf /var/lib/apt/lists/*

# ✅ Copy dependency files first (better cache usage)
COPY package*.json ./
RUN npm ci
COPY . .

# ✅ Use specific image versions
FROM node:18.17.0-alpine3.18

# ✅ Use non-root user
RUN addgroup -S app && adduser -S app -G app
USER app

# ✅ Use COPY instead of ADD (unless you need tar extraction)
COPY ./src /app/src

# ✅ Set WORKDIR instead of cd in RUN
WORKDIR /app

# ✅ Use exec form for CMD/ENTRYPOINT
CMD ["node", "server.js"]

# ✅ Always set ENV for non-interactive installs
ENV DEBIAN_FRONTEND=noninteractive
```

---

## 9. Docker Compose

### docker-compose.yml Structure

```yaml
# docker-compose.yml

version: "3.9"   # Compose file format version

services:

  # ─── Web Application ───────────────────────────
  web:
    image: nginx:alpine
    # Or build from Dockerfile:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
      target: production          # Multi-stage target
    container_name: my-web
    ports:
      - "8080:80"
      - "443:443"
    environment:
      - APP_ENV=production
      - DB_HOST=db
    env_file:
      - .env
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - static_files:/var/www/static
    networks:
      - frontend
      - backend
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    labels:
      - "app=web"
      - "env=prod"
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: "0.50"
          memory: 512M
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    extra_hosts:
      - "myhost:192.168.1.10"
    dns:
      - 8.8.8.8
    ulimits:
      nofile:
        soft: 20000
        hard: 40000

  # ─── Database ──────────────────────────────────
  db:
    image: postgres:15-alpine
    container_name: my-db
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

  # ─── Redis Cache ───────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: my-redis
    command: redis-server --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - backend
    restart: unless-stopped

  # ─── Background Worker ─────────────────────────
  worker:
    build: .
    command: python worker.py
    volumes:
      - .:/app
    environment:
      - DATABASE_URL=postgresql://postgres:secret@db/myapp
    depends_on:
      - db
      - redis
    networks:
      - backend
    restart: on-failure

# ─── Named Volumes ───────────────────────────────
volumes:
  db_data:
    driver: local
  redis_data:
    driver: local
  static_files:
    external: true       # Must exist outside compose

# ─── Networks ────────────────────────────────────
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true       # No external internet access

# ─── Secrets ─────────────────────────────────────
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Docker Compose Commands

```bash
# Start services
docker compose up
docker compose up -d                    # Detached
docker compose up --build               # Rebuild before start
docker compose up --force-recreate      # Recreate containers
docker compose up --no-deps web         # Start only web, skip deps
docker compose up web db                # Start specific services

# Stop services
docker compose down
docker compose down -v                  # Remove volumes too
docker compose down --rmi all           # Remove images too
docker compose down --remove-orphans    # Remove orphan containers

docker compose stop                     # Stop (keep containers)
docker compose start                    # Start stopped services
docker compose restart
docker compose restart web              # Restart specific service

# Logs
docker compose logs
docker compose logs -f                  # Follow all
docker compose logs -f web              # Follow specific service
docker compose logs --tail=50 web

# Exec and run
docker compose exec web bash
docker compose exec -u root web bash
docker compose run --rm web npm test    # One-off container

# Build
docker compose build
docker compose build web                # Build specific service
docker compose build --no-cache

# Status
docker compose ps
docker compose top
docker compose port web 80

# Scale
docker compose up -d --scale web=3

# Config validation
docker compose config
docker compose config --services
docker compose config --volumes

# Multiple compose files
docker compose -f docker-compose.yml -f docker-compose.prod.yml up

# Use different project name
docker compose -p myproject up

# Pull all images
docker compose pull

# Push all images
docker compose push
```

### Override Files

```yaml
# docker-compose.override.yml (auto-loaded with docker-compose.yml)
# Used for local development overrides

services:
  web:
    build:
      context: .
    volumes:
      - .:/app                      # Live code reload
    environment:
      - DEBUG=true
    ports:
      - "3000:3000"

  db:
    ports:
      - "5432:5432"                 # Expose DB port locally
```

---

## 10. Registry & Image Sharing

```bash
# Docker Hub
docker login
docker login -u username
docker logout

# Tag for registry
docker tag myapp username/myapp:latest
docker tag myapp username/myapp:1.0.0

# Push to Docker Hub
docker push username/myapp:latest
docker push username/myapp:1.0.0

# Pull from Docker Hub
docker pull username/myapp:latest

# Private registry
docker login myregistry.com:5000
docker tag myapp myregistry.com:5000/myapp:latest
docker push myregistry.com:5000/myapp:latest
docker pull myregistry.com:5000/myapp:latest

# Run local registry
docker run -d -p 5000:5000 --name registry registry:2
docker tag myapp localhost:5000/myapp
docker push localhost:5000/myapp
docker pull localhost:5000/myapp

# Registry with persistent storage
docker run -d -p 5000:5000 \
  --name registry \
  -v registry_data:/var/lib/registry \
  registry:2

# List images in local registry
curl http://localhost:5000/v2/_catalog
curl http://localhost:5000/v2/myapp/tags/list

# GitHub Container Registry
docker login ghcr.io -u USERNAME --password-stdin <<< $GITHUB_TOKEN
docker tag myapp ghcr.io/username/myapp:latest
docker push ghcr.io/username/myapp:latest

# Amazon ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com
docker tag myapp 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Google Container Registry
gcloud auth configure-docker
docker tag myapp gcr.io/project-id/myapp:latest
docker push gcr.io/project-id/myapp:latest

# Image digest (immutable reference)
docker pull nginx@sha256:abc123...
docker images --digests nginx

# Multi-arch manifest
docker manifest inspect nginx
docker manifest inspect --verbose ubuntu
```

---

## 11. Docker Buildx & Multi-Platform

```bash
# List builders
docker buildx ls

# Create a new builder
docker buildx create --name mybuilder --use
docker buildx create --name mybuilder --driver docker-container --use

# Inspect builder
docker buildx inspect mybuilder
docker buildx inspect --bootstrap mybuilder

# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

# Build and push multi-platform
docker buildx build \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  -t username/myapp:latest \
  --push \
  .

# Build and load to local Docker
docker buildx build --platform linux/amd64 -t myapp --load .

# Build with cache
docker buildx build \
  --cache-from type=registry,ref=username/myapp:cache \
  --cache-to type=registry,ref=username/myapp:cache,mode=max \
  -t username/myapp:latest \
  --push .

# Bake (build multiple images from config)
docker buildx bake
docker buildx bake --file docker-bake.hcl

# Remove builder
docker buildx rm mybuilder

# Use BuildKit features (inline cache, secrets)
DOCKER_BUILDKIT=1 docker build \
  --secret id=mysecret,src=./secret.txt \
  -t myapp .
```

```dockerfile
# Use secret in Dockerfile (BuildKit)
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret

# SSH forwarding in build
RUN --mount=type=ssh git clone git@github.com:private/repo.git

# Cache mount (speed up package installs)
RUN --mount=type=cache,target=/root/.npm npm install
RUN --mount=type=cache,target=/var/cache/apt apt-get update && apt-get install -y curl
```

---

## 12. Resource Management & Limits

```bash
# Memory limits
docker run --memory="512m" nginx               # Hard limit
docker run --memory="512m" --memory-swap="1g" nginx  # Swap included
docker run --memory-reservation="256m" nginx   # Soft limit
docker run --memory-swappiness=0 nginx         # Disable swap usage

# CPU limits
docker run --cpus="1.5" nginx                  # 1.5 CPU cores
docker run --cpu-shares=512 nginx              # Relative weight (default 1024)
docker run --cpuset-cpus="0,1" nginx           # Pin to CPU 0 and 1
docker run --cpu-period=100000 \
           --cpu-quota=50000 nginx             # 50% of one CPU

# Disk I/O limits
docker run --device-read-bps /dev/sda:1mb nginx
docker run --device-write-bps /dev/sda:1mb nginx
docker run --device-read-iops /dev/sda:1000 nginx
docker run --device-write-iops /dev/sda:1000 nginx
docker run --blkio-weight=500 nginx            # Block I/O weight (10-1000)

# Kernel limits (ulimits)
docker run --ulimit nofile=1024:1024 nginx
docker run --ulimit nproc=3 nginx

# PID limit
docker run --pids-limit 200 nginx

# Update resource limits on running container
docker update --memory 1g mycontainer
docker update --cpus 2 mycontainer
docker update --memory-swap -1 mycontainer    # Unlimited swap

# Check container resource usage
docker stats --no-stream mycontainer
docker stats --format "{{.Name}}: CPU={{.CPUPerc}} MEM={{.MemUsage}}"

# OOM (Out Of Memory) killer behavior
docker run --oom-kill-disable nginx           # Disable OOM killer (risky)
docker run --oom-score-adj=-500 nginx         # Reduce likelihood of OOM kill
```

---

## 13. Logging & Monitoring

```bash
# View logs
docker logs mycontainer
docker logs -f mycontainer                    # Follow
docker logs --tail 100 mycontainer            # Last 100 lines
docker logs --since "2024-01-01T00:00:00" mycontainer
docker logs --since 30m mycontainer           # Last 30 minutes
docker logs --until 1h mycontainer            # Older than 1 hour
docker logs -t mycontainer                    # Include timestamps

# Logging drivers
docker run --log-driver none myapp                         # No logs
docker run --log-driver json-file myapp                    # Default (JSON file)
docker run --log-driver syslog myapp                       # System syslog
docker run --log-driver journald myapp                     # systemd journal
docker run --log-driver fluentd myapp                      # Fluentd
docker run --log-driver awslogs myapp                      # AWS CloudWatch
docker run --log-driver splunk myapp                       # Splunk
docker run --log-driver gelf myapp                         # Graylog GELF

# json-file options
docker run \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  --log-opt compress=true \
  nginx

# Syslog options
docker run \
  --log-driver syslog \
  --log-opt syslog-address=tcp://192.168.1.3:514 \
  --log-opt tag="myapp" \
  nginx

# Check log driver of a container
docker inspect --format='{{.HostConfig.LogConfig.Type}}' mycontainer

# Real-time events
docker events
docker events --filter event=start
docker events --filter container=mycontainer
docker events --filter image=nginx
docker events --since 1h

# Stats monitoring
docker stats
docker stats --no-stream
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# Container processes
docker top mycontainer
docker top mycontainer aux

# Container resource inspection
docker inspect --format='{{.HostConfig.Memory}}' mycontainer
docker inspect --format='{{.HostConfig.NanoCpus}}' mycontainer
```

---

## 14. Security

```bash
# Run as non-root user
docker run -u 1000 myapp
docker run -u nobody myapp
docker run --user $(id -u):$(id -g) myapp

# Read-only filesystem
docker run --read-only myapp
docker run --read-only --tmpfs /tmp --tmpfs /var/run myapp

# Drop all capabilities, add only what's needed
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp

# Linux capabilities
docker run --cap-add SYS_PTRACE myapp     # Debugging
docker run --cap-add NET_ADMIN myapp      # Network config
docker run --cap-drop SETUID myapp        # Drop setuid

# Security options
docker run --security-opt no-new-privileges myapp
docker run --security-opt seccomp=seccomp-profile.json myapp
docker run --security-opt apparmor=myprofile myapp

# Seccomp profile (default restricts ~300 syscalls)
docker run --security-opt seccomp=unconfined myapp   # No seccomp

# Namespaces
docker run --userns=host myapp            # Disable user namespace remapping
docker run --pid=host myapp              # Share host PID namespace
docker run --ipc=host myapp              # Share host IPC namespace
docker run --network=host myapp          # Share host network

# Limit /proc visibility
docker run --read-only -v /proc/bus:/proc/bus:ro myapp

# Scan image for vulnerabilities
docker scout cves myimage:latest
docker scout recommendations myimage:latest

# Check image with Trivy (3rd party)
trivy image myimage:latest

# Docker Content Trust (signed images)
export DOCKER_CONTENT_TRUST=1
docker pull nginx     # Only pulls signed images

# Secrets (don't bake secrets into images)
# Use BuildKit secrets:
docker build --secret id=mysecret,src=./secret.txt -t myapp .

# Docker Swarm secrets
docker secret create db_password password.txt
docker service create --secret db_password myapp

# Environment secrets (avoid where possible — visible in inspect)
docker run -e SECRET_KEY=abc123 myapp

# Better: Use secret files or external secret managers
docker run -v /run/secrets:/run/secrets:ro myapp
```

---

## 15. Docker Contexts

```bash
# List contexts
docker context ls

# Show current context
docker context show

# Create context (remote Docker host)
docker context create myremote \
  --docker "host=ssh://user@remote-host"

# Create context with TLS
docker context create myremote \
  --docker "host=tcp://remote:2376,ca=/certs/ca.pem,cert=/certs/cert.pem,key=/certs/key.pem"

# Switch context
docker context use myremote
docker context use default

# Run single command on another context
docker --context myremote ps

# Inspect context
docker context inspect myremote

# Remove context
docker context rm myremote

# Export / import context
docker context export myremote --output myremote.dockercontext
docker context import myremote myremote.dockercontext
```

---

## 16. Docker Swarm

```bash
# Initialize Swarm
docker swarm init
docker swarm init --advertise-addr 192.168.1.10

# Get join tokens
docker swarm join-token worker
docker swarm join-token manager

# Join Swarm
docker swarm join --token SWMTKN-1-xxx 192.168.1.10:2377

# Leave Swarm
docker swarm leave
docker swarm leave --force     # On manager

# Node management
docker node ls
docker node inspect self
docker node inspect node1
docker node update --availability drain node1    # Stop new tasks
docker node update --availability active node1
docker node update --role manager node1
docker node update --label-add region=us-east node1
docker node rm node1

# Services
docker service create --name web --replicas 3 -p 80:80 nginx
docker service create \
  --name myapp \
  --replicas 3 \
  --update-parallelism 1 \
  --update-delay 10s \
  --rollback-parallelism 1 \
  --constraint "node.role==worker" \
  --constraint "node.labels.region==us-east" \
  --secret db_password \
  --env DB_HOST=db \
  --network myoverlay \
  --mount type=volume,source=myvolume,target=/data \
  --resource-limit-cpu 0.5 \
  --resource-limit-memory 512m \
  -p 8080:80 \
  myapp:latest

docker service ls
docker service ps myapp
docker service inspect myapp
docker service logs myapp
docker service logs -f myapp

# Update service
docker service update --image myapp:2.0 myapp
docker service update --replicas 5 myapp
docker service update --env-add NEW_VAR=value myapp
docker service update --publish-add 443:443 myapp
docker service update --rollback myapp     # Rollback last update
docker service scale myapp=5               # Scale

# Remove service
docker service rm myapp

# Stacks (multi-service compose-like deployment)
docker stack deploy -c docker-compose.yml mystack
docker stack ls
docker stack ps mystack
docker stack services mystack
docker stack rm mystack

# Secrets
docker secret create db_password ./password.txt
docker secret ls
docker secret inspect db_password
docker secret rm db_password

# Configs
docker config create nginx_conf ./nginx.conf
docker config ls
docker config inspect nginx_conf
docker config rm nginx_conf

# Overlay network for Swarm
docker network create \
  --driver overlay \
  --attachable \
  --subnet 10.10.0.0/16 \
  myoverlay

# Rolling updates
docker service update \
  --update-parallelism 2 \
  --update-delay 30s \
  --update-failure-action rollback \
  --image myapp:2.0 \
  myapp

# Health-based updates
docker service update \
  --update-monitor 60s \
  --update-max-failure-ratio 0.1 \
  --image myapp:2.0 \
  myapp
```

---

## 17. Pruning & Cleanup

```bash
# Remove stopped containers
docker container prune
docker container prune --force

# Remove unused images
docker image prune             # Dangling images only
docker image prune -a          # All unused images
docker image prune --filter "until=24h"

# Remove unused volumes
docker volume prune
docker volume prune --force

# Remove unused networks
docker network prune

# Remove all unused objects (containers, images, volumes, networks)
docker system prune
docker system prune -a                          # Include unused images
docker system prune -a --volumes                # Also volumes
docker system prune -a --volumes --force        # No confirmation

# Filter pruning
docker container prune --filter "until=24h"
docker image prune -a --filter "until=72h"
docker image prune -a --filter "label=env=dev"

# Remove all containers (stop + remove)
docker stop $(docker ps -q) && docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q) --force

# Remove all volumes
docker volume rm $(docker volume ls -q)

# Nuclear option — remove EVERYTHING
docker system prune -a --volumes --force

# How much space Docker is using
docker system df
docker system df -v    # Verbose, shows each object
```

---

## 18. Performance & Debugging

```bash
# Analyze image size and layers
docker history myimage
docker history --no-trunc --format "{{.Size}}\t{{.CreatedBy}}" myimage

# Dive tool — explore image layers interactively (3rd party)
dive myimage:latest

# Inspect any object
docker inspect mycontainer
docker inspect myimage
docker inspect myvolume
docker inspect mynetwork

# Format inspect output
docker inspect --format='{{.State.Status}}' mycontainer
docker inspect --format='{{.NetworkSettings.IPAddress}}' mycontainer
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mycontainer

# Debug a running container
docker exec -it mycontainer sh
docker exec -it mycontainer bash
docker exec -it mycontainer top
docker exec -it mycontainer netstat -tlnp
docker exec -it mycontainer cat /etc/hosts
docker exec -it mycontainer env

# Debug a stopped container
docker start mycontainer
docker exec -it mycontainer sh

# Attach to a crashed container for debugging
docker run -it --entrypoint bash myimage

# Copy config/logs out of a container
docker cp mycontainer:/etc/nginx/nginx.conf ./nginx.conf
docker cp mycontainer:/var/log/app.log ./app.log

# Check filesystem changes
docker diff mycontainer

# Profile container networking
docker run --network container:mycontainer \
  nicolaka/netshoot \
  iperf3 -c targethost

# Strace inside container (requires SYS_PTRACE)
docker run --cap-add SYS_PTRACE myapp
docker exec mycontainer strace -p 1

# Run lightweight debug container on same namespace
docker run -it --pid=container:mycontainer \
           --net=container:mycontainer \
           --cap-add SYS_PTRACE busybox sh

# Check Docker daemon logs
journalctl -u docker.service
journalctl -u docker.service -f         # Follow
journalctl -u docker.service --since today

# Remote debug Docker daemon
dockerd --debug

# BuildKit debug
BUILDKIT_PROGRESS=plain docker build .
```

---

## 19. Advanced Features

### Docker Extensions & Plugins

```bash
# Volume plugins
docker plugin install vieux/sshfs
docker plugin ls
docker plugin enable vieux/sshfs
docker plugin disable vieux/sshfs
docker plugin rm vieux/sshfs

# Use volume plugin
docker volume create -d vieux/sshfs \
  -o sshcmd=user@host:/path \
  -o password=secret \
  sshvolume

# Log plugins
docker plugin install grafana/loki-docker-driver:latest \
  --alias loki --grant-all-permissions

docker run \
  --log-driver=loki \
  --log-opt loki-url="http://loki:3100/loki/api/v1/push" \
  nginx
```

### Health Checks

```bash
# Runtime healthcheck
docker run \
  --health-cmd='curl -f http://localhost/ || exit 1' \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=10s \
  nginx

# Check health status
docker inspect --format='{{.State.Health.Status}}' mycontainer
docker inspect --format='{{json .State.Health}}' mycontainer | jq

# Wait for healthy
until [ "$(docker inspect -f '{{.State.Health.Status}}' mycontainer)" = "healthy" ]; do
  echo "Waiting for container to be healthy..."
  sleep 2
done
```

### Labels & Filtering

```bash
# Add labels
docker run --label app=backend --label env=prod nginx
docker run --label-file labels.txt nginx

# Filter by label
docker ps --filter label=app=backend
docker images --filter label=env=prod
docker network ls --filter label=project=myapp

# Inspect label
docker inspect --format='{{json .Config.Labels}}' mycontainer
```

### Docker Init (Proper PID 1)

```bash
# Use tini as init (handles zombie processes)
docker run --init myapp

# Or in Dockerfile
FROM node:18-alpine
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]
```

### Environment Patterns

```bash
# .env file (loaded by Compose automatically)
# .env
POSTGRES_USER=admin
POSTGRES_PASSWORD=secret
APP_PORT=3000

# Reference in compose
services:
  db:
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

# Variable substitution
docker compose --env-file .env.prod up

# Pass host env vars to container
docker run -e HOME myapp          # Pass HOST's $HOME
docker run -e HOST_HOME=$HOME myapp

# Use secrets file (safer than env vars)
docker run -v $(pwd)/secrets:/run/secrets:ro myapp
```

### Docker API

```bash
# Docker exposes REST API via socket

# List containers via API
curl --unix-socket /var/run/docker.sock http://localhost/containers/json

# Start container via API
curl --unix-socket /var/run/docker.sock \
  -X POST http://localhost/containers/mycontainer/start

# Remote API (TCP — secure with TLS in production)
docker -H tcp://remote:2376 ps

# Enable Docker API (add to /etc/docker/daemon.json)
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"]
}
```

### daemon.json Configuration

```json
// /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-ulimits": {
    "nofile": { "Hard": 64000, "Name": "nofile", "Soft": 64000 }
  },
  "storage-driver": "overlay2",
  "insecure-registries": ["myregistry.local:5000"],
  "registry-mirrors": ["https://mirror.gcr.io"],
  "live-restore": true,
  "userland-proxy": false,
  "experimental": true,
  "metrics-addr": "0.0.0.0:9323",
  "max-concurrent-downloads": 10,
  "max-concurrent-uploads": 5,
  "dns": ["8.8.8.8", "8.8.4.4"],
  "data-root": "/mnt/docker-data"
}
```

```bash
# Reload daemon config
sudo systemctl reload docker
# Or
sudo kill -SIGHUP $(pidof dockerd)
```

---

## Quick Reference Cheatsheet

| Task | Command |
|------|---------|
| Build image | `docker build -t myapp:latest .` |
| Run container | `docker run -d -p 8080:80 --name myapp myapp:latest` |
| Shell in container | `docker exec -it myapp bash` |
| View logs | `docker logs -f myapp` |
| Stop container | `docker stop myapp` |
| Remove container | `docker rm myapp` |
| List containers | `docker ps -a` |
| List images | `docker images` |
| Remove image | `docker rmi myapp:latest` |
| Push image | `docker push username/myapp:latest` |
| Pull image | `docker pull nginx:alpine` |
| Compose up | `docker compose up -d --build` |
| Compose down | `docker compose down -v` |
| Compose logs | `docker compose logs -f` |
| Prune all | `docker system prune -a --volumes` |
| Disk usage | `docker system df` |
| Resource stats | `docker stats` |
| Swarm init | `docker swarm init` |
| Deploy stack | `docker stack deploy -c compose.yml mystack` |

---

*Docker version reference: Commands work on Docker Engine 20.10+. Compose commands use `docker compose` (V2 plugin). Legacy `docker-compose` (V1 Python) uses the same syntax with a hyphen.*
