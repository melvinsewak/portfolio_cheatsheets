# Docker Cheatsheet

Welcome to the Docker cheatsheet! This guide will help you quickly master containerization with Docker.

## Overview

Docker is a platform for developing, shipping, and running applications in containers. Containers package software with all its dependencies, ensuring consistent behavior across different environments. Docker has revolutionized modern software development by making applications portable, scalable, and efficient.

## Contents

- [Beginner.md](Beginner.md) - Installation, images, containers, basic commands, and Dockerfiles
- [Intermediate.md](Intermediate.md) - Networking, volumes, Docker Compose, building images, and registries
- [Advanced.md](Advanced.md) - Multi-stage builds, orchestration, security, optimization, and troubleshooting

## Quick Reference

### Basic Commands
```bash
# Pull an image
docker pull nginx:latest

# Run a container
docker run -d -p 8080:80 --name mynginx nginx

# List running containers
docker ps

# Stop a container
docker stop mynginx

# Remove a container
docker rm mynginx
```

### Simple Dockerfile
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Docker Compose Example
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
```

### Key Features

- **Portability**: Run anywhere - dev laptop, test server, production cloud
- **Isolation**: Each container runs independently with its own filesystem
- **Efficiency**: Lightweight compared to virtual machines
- **Scalability**: Easily scale applications horizontally
- **Version Control**: Images are versioned and reproducible

## Getting Started

### Install Docker

**Linux:**
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

**macOS/Windows:**
Download Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop)

### Verify Installation
```bash
docker --version
docker run hello-world
```

### First Container
```bash
# Run an nginx web server
docker run -d -p 8080:80 nginx

# Visit http://localhost:8080 in your browser
```

## Docker Architecture

```
┌─────────────────────────────────────┐
│         Docker Client (CLI)         │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│          Docker Daemon              │
│  ┌──────────────────────────────┐   │
│  │       Containers             │   │
│  │  ┌────┐  ┌────┐  ┌────┐     │   │
│  │  │ C1 │  │ C2 │  │ C3 │     │   │
│  │  └────┘  └────┘  └────┘     │   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │         Images               │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│        Docker Registry (Hub)        │
└─────────────────────────────────────┘
```

## Common Workflows

### Development Workflow
```bash
# 1. Create Dockerfile
# 2. Build image
docker build -t myapp:dev .

# 3. Run container
docker run -p 3000:3000 -v $(pwd):/app myapp:dev

# 4. Test and iterate
```

### Production Workflow
```bash
# 1. Build optimized image
docker build -t myapp:1.0.0 .

# 2. Tag for registry
docker tag myapp:1.0.0 username/myapp:1.0.0

# 3. Push to registry
docker push username/myapp:1.0.0

# 4. Deploy
docker pull username/myapp:1.0.0
docker run -d -p 80:3000 username/myapp:1.0.0
```

## Essential Concepts

- **Image**: Read-only template with application code and dependencies
- **Container**: Running instance of an image
- **Dockerfile**: Script to build Docker images
- **Volume**: Persistent data storage for containers
- **Network**: Communication between containers
- **Registry**: Repository for Docker images (Docker Hub, private registries)

## Quick Tips

✅ **DO:**
- Use official images as base images
- Keep images small and focused
- Use .dockerignore to exclude unnecessary files
- Tag images with meaningful versions
- Run containers as non-root users

❌ **DON'T:**
- Store secrets in images
- Run everything as root
- Use `:latest` tag in production
- Include development tools in production images
- Ignore security updates

## Additional Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Samples](https://github.com/docker/samples)
- [Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Play with Docker](https://labs.play-with-docker.com/) - Free online playground
