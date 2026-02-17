# Docker Beginner Cheatsheet

## Installation

### Linux (Ubuntu/Debian)
```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add user to docker group (optional - run docker without sudo)
sudo usermod -aG docker $USER
newgrp docker
```

### macOS / Windows
```bash
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop

# After installation, verify:
docker --version
docker compose version
```

### Verify Installation
```bash
# Check version
docker --version

# Run test container
docker run hello-world

# Check Docker info
docker info
```

## Understanding Docker Images

### What is an Image?
An image is a read-only template containing:
- Application code
- Runtime environment
- System libraries
- Dependencies
- Configuration files

### Working with Images

#### Pull Images from Docker Hub
```bash
# Pull latest version
docker pull ubuntu

# Pull specific version
docker pull ubuntu:22.04

# Pull from official registry
docker pull nginx:1.25-alpine

# Search for images
docker search mysql
```

#### List Images
```bash
# List all images
docker images

# List with details
docker images -a

# Filter images
docker images ubuntu

# Show image sizes
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

#### Remove Images
```bash
# Remove single image
docker rmi ubuntu:22.04

# Remove by image ID
docker rmi abc123def456

# Remove multiple images
docker rmi nginx ubuntu redis

# Force remove (even if containers exist)
docker rmi -f image_name

# Remove unused images
docker image prune

# Remove all unused images
docker image prune -a
```

## Working with Containers

### What is a Container?
A container is a runnable instance of an image. It includes:
- The image
- Execution environment
- Standard instruction set

### Basic Container Commands

#### Run Containers
```bash
# Run container (foreground)
docker run ubuntu echo "Hello Docker"

# Run container (detached/background)
docker run -d nginx

# Run with custom name
docker run -d --name mynginx nginx

# Run interactively
docker run -it ubuntu bash

# Run with automatic removal after exit
docker run --rm ubuntu echo "Temporary container"

# Run specific version
docker run -d nginx:1.25-alpine
```

#### List Containers
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Show only container IDs
docker ps -q

# Filter containers
docker ps --filter "name=mynginx"
docker ps --filter "status=exited"

# Show last created container
docker ps -l

# Custom format
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

#### Stop Containers
```bash
# Stop container gracefully (SIGTERM)
docker stop mynginx

# Stop container forcefully (SIGKILL)
docker kill mynginx

# Stop multiple containers
docker stop container1 container2 container3

# Stop all running containers
docker stop $(docker ps -q)

# Stop with timeout
docker stop -t 30 mynginx
```

#### Start and Restart Containers
```bash
# Start stopped container
docker start mynginx

# Restart running container
docker restart mynginx

# Start and attach to container
docker start -a mynginx

# Start in interactive mode
docker start -i myubuntu
```

#### Remove Containers
```bash
# Remove stopped container
docker rm mynginx

# Force remove running container
docker rm -f mynginx

# Remove multiple containers
docker rm container1 container2

# Remove all stopped containers
docker container prune

# Remove all containers (stopped and running)
docker rm -f $(docker ps -aq)
```

#### Interact with Running Containers
```bash
# Execute command in running container
docker exec mynginx ls /usr/share/nginx/html

# Open interactive shell
docker exec -it mynginx bash
docker exec -it myubuntu sh

# Run as specific user
docker exec -u root -it mynginx bash

# Execute with environment variable
docker exec -e VAR=value mynginx env
```

#### View Container Information
```bash
# View container logs
docker logs mynginx

# Follow logs in real-time
docker logs -f mynginx

# Show last N lines
docker logs --tail 50 mynginx

# Show logs with timestamps
docker logs -t mynginx

# View container details
docker inspect mynginx

# View resource usage stats
docker stats mynginx

# View running processes
docker top mynginx
```

## Port Mapping

### Understanding Ports
Map container ports to host machine ports to access services.

### Port Mapping Examples
```bash
# Map port 8080 on host to port 80 in container
docker run -d -p 8080:80 nginx

# Map to random host port
docker run -d -P nginx

# Map specific host IP
docker run -d -p 127.0.0.1:8080:80 nginx

# Map multiple ports
docker run -d -p 8080:80 -p 8443:443 nginx

# Map UDP port
docker run -d -p 53:53/udp dns-server

# View port mappings
docker port mynginx
```

### Access Container Services
```bash
# After running: docker run -d -p 8080:80 --name web nginx
# Access via: http://localhost:8080

# Check port binding
docker ps
# or
netstat -tlnp | grep 8080
```

## Volume Basics

### Understanding Volumes
Volumes persist data generated by and used by containers.

### Types of Mounts

#### Bind Mounts (Host directory)
```bash
# Mount host directory to container
docker run -d -v /path/on/host:/path/in/container nginx

# Mount current directory
docker run -d -v $(pwd):/app node:18

# Read-only mount
docker run -d -v /host/path:/container/path:ro nginx

# Example: Serve local HTML files
docker run -d -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html nginx
```

#### Named Volumes (Docker managed)
```bash
# Create named volume
docker volume create mydata

# Use named volume
docker run -d -v mydata:/data postgres

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata

# Remove unused volumes
docker volume prune
```

### Volume Examples
```bash
# MySQL with persistent data
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

# Node.js app with source code mount
docker run -d \
  --name myapp \
  -p 3000:3000 \
  -v $(pwd):/app \
  -v /app/node_modules \
  node:18

# Nginx with custom config
docker run -d \
  --name webserver \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx
```

## Dockerfile Basics

### What is a Dockerfile?
A text file containing instructions to build a Docker image.

### Basic Dockerfile Structure
```dockerfile
# Use official base image
FROM node:18-alpine

# Set metadata
LABEL maintainer="you@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy files
COPY package*.json ./

# Run commands
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Define default command
CMD ["node", "server.js"]
```

### Common Dockerfile Instructions

#### FROM - Base Image
```dockerfile
# Official image
FROM ubuntu:22.04

# Official language runtime
FROM node:18-alpine
FROM python:3.11-slim

# Scratch (empty base)
FROM scratch
```

#### WORKDIR - Set Working Directory
```dockerfile
# Set working directory (creates if doesn't exist)
WORKDIR /app

# All subsequent commands run from here
WORKDIR /usr/src/myapp
```

#### COPY - Copy Files
```dockerfile
# Copy single file
COPY package.json /app/

# Copy multiple files
COPY package.json package-lock.json ./

# Copy directory
COPY src/ /app/src/

# Copy everything
COPY . .

# Copy with ownership
COPY --chown=node:node . /app
```

#### ADD - Copy and Extract
```dockerfile
# Similar to COPY but can extract archives
ADD archive.tar.gz /app/

# Download from URL (not recommended)
ADD https://example.com/file.tar.gz /app/

# Note: Prefer COPY over ADD unless you need extraction
```

#### RUN - Execute Commands
```dockerfile
# Install packages
RUN apt-get update && apt-get install -y curl

# Install Node.js dependencies
RUN npm install

# Multiple commands (each RUN creates a layer)
RUN apt-get update
RUN apt-get install -y python3

# Combine commands to reduce layers
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

#### CMD - Default Command
```dockerfile
# Exec form (preferred)
CMD ["node", "server.js"]
CMD ["python", "app.py"]

# Shell form
CMD node server.js

# Only last CMD instruction is used
CMD ["npm", "start"]
```

#### ENTRYPOINT - Configure Container Executable
```dockerfile
# Make container run as executable
ENTRYPOINT ["node"]
CMD ["server.js"]

# Can override CMD but not ENTRYPOINT
# docker run image app.js  -> runs: node app.js

# Use with script
ENTRYPOINT ["./docker-entrypoint.sh"]
```

#### EXPOSE - Document Ports
```dockerfile
# Document which port app listens on
EXPOSE 3000

# Multiple ports
EXPOSE 3000 8080

# Note: Doesn't actually publish port, just documentation
# Still need -p flag: docker run -p 3000:3000 image
```

#### ENV - Environment Variables
```dockerfile
# Set environment variable
ENV NODE_ENV=production

# Multiple variables
ENV NODE_ENV=production \
    PORT=3000 \
    API_URL=https://api.example.com

# Use in commands
ENV APP_HOME=/app
WORKDIR $APP_HOME
```

#### ARG - Build Arguments
```dockerfile
# Define build argument
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

ARG BUILD_DATE
ARG VERSION=1.0.0

LABEL build-date=$BUILD_DATE
LABEL version=$VERSION

# Pass during build:
# docker build --build-arg NODE_VERSION=20 --build-arg BUILD_DATE=$(date) .
```

### Complete Dockerfile Examples

#### Node.js Application
```dockerfile
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

# Start application
CMD ["node", "server.js"]
```

#### Python Flask Application
```dockerfile
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 5000

# Run application
CMD ["python", "app.py"]
```

#### Static Website (Nginx)
```dockerfile
FROM nginx:alpine

# Copy custom nginx config (optional)
COPY nginx.conf /etc/nginx/nginx.conf

# Copy website files
COPY html/ /usr/share/nginx/html/

# Expose port
EXPOSE 80

# nginx image already has CMD to start nginx
```

### Building Images

#### Build Commands
```bash
# Build from Dockerfile in current directory
docker build -t myapp:1.0 .

# Build with different Dockerfile
docker build -t myapp:1.0 -f Dockerfile.prod .

# Build with build arguments
docker build --build-arg VERSION=2.0 -t myapp:2.0 .

# Build without cache
docker build --no-cache -t myapp:1.0 .

# Build with tag and no cache
docker build -t myapp:latest --no-cache .

# Build and show all output
docker build -t myapp:1.0 --progress=plain .
```

#### .dockerignore File
```
# .dockerignore - exclude files from build context
node_modules
npm-debug.log
.env
.git
.gitignore
*.md
.vscode
.idea
dist
coverage
.cache
.DS_Store
*.log
```

## Environment Variables

### Setting Environment Variables
```bash
# Single variable
docker run -e NODE_ENV=production myapp

# Multiple variables
docker run \
  -e NODE_ENV=production \
  -e PORT=3000 \
  -e DB_HOST=localhost \
  myapp

# From file
docker run --env-file .env myapp

# Example .env file:
# NODE_ENV=production
# PORT=3000
# DB_HOST=db.example.com
# DB_USER=admin
```

### Using in Dockerfile
```dockerfile
# Set default values
ENV NODE_ENV=development
ENV PORT=3000

# Use in commands
RUN echo "Environment: $NODE_ENV"

# Can override at runtime:
# docker run -e NODE_ENV=production myapp
```

## Container Lifecycle Management

### Container States
```
Created → Running → Paused → Stopped → Removed
```

### Lifecycle Commands
```bash
# Create container without starting
docker create --name mycontainer nginx

# Start created container
docker start mycontainer

# Pause container (freeze)
docker pause mycontainer

# Unpause container
docker unpause mycontainer

# Stop container (SIGTERM, then SIGKILL after timeout)
docker stop mycontainer

# Kill container immediately (SIGKILL)
docker kill mycontainer

# Restart container
docker restart mycontainer

# Remove container
docker rm mycontainer

# Wait for container to stop
docker wait mycontainer
```

### Automatic Restart Policies
```bash
# Always restart
docker run -d --restart=always nginx

# Restart unless manually stopped
docker run -d --restart=unless-stopped nginx

# Restart on failure only
docker run -d --restart=on-failure nginx

# Restart on failure with max retries
docker run -d --restart=on-failure:5 nginx

# Update restart policy on running container
docker update --restart=always mycontainer
```

## Resource Limits

### Memory Limits
```bash
# Limit memory to 512MB
docker run -d --memory=512m nginx

# Memory with swap
docker run -d --memory=512m --memory-swap=1g nginx

# Soft limit (reservation)
docker run -d --memory-reservation=256m nginx
```

### CPU Limits
```bash
# Limit CPU shares (relative weight)
docker run -d --cpu-shares=512 nginx

# Limit to specific CPU cores
docker run -d --cpuset-cpus="0,1" nginx

# Limit CPU percentage
docker run -d --cpus=1.5 nginx
```

## Do's and Don'ts

### ✅ DO:

1. **Use Official Images**
```dockerfile
FROM node:18-alpine
FROM python:3.11-slim
FROM nginx:alpine
```

2. **Tag Your Images**
```bash
docker build -t myapp:1.0.0 .
docker build -t myapp:1.0.0 -t myapp:latest .
```

3. **Use .dockerignore**
```
node_modules
.git
*.md
.env
```

4. **Clean Up Regularly**
```bash
docker system prune
docker image prune
docker container prune
docker volume prune
```

5. **Use Named Volumes for Data**
```bash
docker run -v mydata:/data postgres
```

6. **Run as Non-Root User**
```dockerfile
RUN useradd -m appuser
USER appuser
```

7. **Use Health Checks**
```dockerfile
HEALTHCHECK CMD curl --fail http://localhost:3000/health || exit 1
```

8. **Check Container Logs**
```bash
docker logs -f mycontainer
```

### ❌ DON'T:

1. **Don't Store Secrets in Images**
```dockerfile
# ❌ Bad
ENV API_KEY=secret123
RUN echo "password=secret" > config

# ✅ Good - use at runtime
docker run -e API_KEY=secret123 myapp
```

2. **Don't Use :latest in Production**
```bash
# ❌ Bad
docker pull nginx:latest

# ✅ Good
docker pull nginx:1.25.3
```

3. **Don't Run as Root**
```dockerfile
# ❌ Bad (default is root)
CMD ["node", "server.js"]

# ✅ Good
USER node
CMD ["node", "server.js"]
```

4. **Don't Create Large Images**
```dockerfile
# ❌ Bad
FROM ubuntu
RUN apt-get update && apt-get install -y nodejs npm

# ✅ Good
FROM node:18-alpine
```

5. **Don't Ignore .dockerignore**
```
# Always create .dockerignore to exclude:
# - node_modules
# - .git
# - build artifacts
# - local configs
```

6. **Don't Install Unnecessary Packages**
```dockerfile
# ❌ Bad
RUN apt-get install -y build-essential git curl wget vim

# ✅ Good
RUN apt-get install -y --no-install-recommends build-essential
```

7. **Don't Forget to Clean Up**
```bash
# ❌ Bad - leaves stopped containers
docker run nginx

# ✅ Good - auto-removes
docker run --rm nginx
```

8. **Don't Expose Unnecessary Ports**
```dockerfile
# Only expose what's needed
EXPOSE 3000
# Don't expose: 22, 3306, 5432 unless required
```

## Common Issues and Solutions

### Issue: Permission Denied
```bash
# Problem: docker: permission denied
# Solution: Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

### Issue: Port Already in Use
```bash
# Problem: Port 8080 already allocated
# Solution: Use different port or stop conflicting container
docker ps
docker stop conflicting-container
# Or use different port
docker run -p 8081:80 nginx
```

### Issue: Container Exits Immediately
```bash
# Problem: Container stops right after starting
# Solution: Check logs
docker logs container-name

# Common cause: No foreground process
# Fix: Ensure CMD/ENTRYPOINT runs foreground process
```

### Issue: Cannot Remove Image
```bash
# Problem: Image in use by container
# Solution: Remove containers first
docker rm $(docker ps -aq --filter ancestor=image-name)
docker rmi image-name
```

## Quick Command Reference

```bash
# Images
docker pull <image>              # Download image
docker images                    # List images
docker rmi <image>               # Remove image
docker build -t <name> .         # Build image

# Containers
docker run <image>               # Create and start container
docker ps                        # List running containers
docker ps -a                     # List all containers
docker stop <container>          # Stop container
docker start <container>         # Start container
docker rm <container>            # Remove container
docker exec -it <container> sh   # Open shell in container

# Logs and Info
docker logs <container>          # View logs
docker logs -f <container>       # Follow logs
docker inspect <container>       # Detailed info
docker stats                     # Resource usage

# Cleanup
docker system prune              # Remove unused resources
docker container prune           # Remove stopped containers
docker image prune              # Remove unused images
docker volume prune             # Remove unused volumes
```
