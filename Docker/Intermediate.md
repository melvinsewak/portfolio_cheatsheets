# Docker Intermediate Cheatsheet

## Docker Networking

### Understanding Docker Networks
Docker networks allow containers to communicate with each other and the outside world.

### Network Drivers

#### Bridge (Default)
Default network driver for containers on a single host.

```bash
# Create bridge network
docker network create mynetwork

# Run container on custom network
docker run -d --name app1 --network mynetwork nginx

# Run another container on same network
docker run -d --name app2 --network mynetwork redis

# Containers can communicate using container names
docker exec app1 ping app2
```

#### Host
Container shares host's network stack (no isolation).

```bash
# Use host network
docker run -d --network host nginx

# Container uses host's IP and ports directly
# No port mapping needed
```

#### None
Disable networking completely.

```bash
# No network access
docker run -d --network none nginx
```

#### Overlay
Multi-host networking for Docker Swarm.

```bash
# Create overlay network (requires Swarm)
docker network create -d overlay my-overlay-net
```

### Network Commands

```bash
# List networks
docker network ls

# Inspect network
docker network inspect mynetwork

# Create network
docker network create mynetwork

# Create network with subnet
docker network create --subnet=172.18.0.0/16 mynetwork

# Create network with custom driver
docker network create -d bridge mybridge

# Connect running container to network
docker network connect mynetwork mycontainer

# Disconnect container from network
docker network disconnect mynetwork mycontainer

# Remove network
docker network rm mynetwork

# Remove unused networks
docker network prune
```

### Network Examples

#### Multi-Container Application
```bash
# Create custom network
docker network create app-network

# Run database
docker run -d \
  --name postgres-db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Run application (can connect to postgres-db by name)
docker run -d \
  --name web-app \
  --network app-network \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://postgres:secret@postgres-db:5432/mydb \
  myapp:latest

# Run admin tool
docker run -it \
  --name pgadmin \
  --network app-network \
  -p 5050:80 \
  dpage/pgadmin4
```

#### Network with Aliases
```bash
# Create network
docker network create mynet

# Run container with network alias
docker run -d \
  --name db1 \
  --network mynet \
  --network-alias database \
  postgres:15

# Other containers can reach it via "database" hostname
docker run -it \
  --network mynet \
  alpine ping database
```

### DNS and Service Discovery

```bash
# Containers on same custom network can resolve each other by name
docker network create mynet
docker run -d --name web --network mynet nginx
docker run -d --name api --network mynet node:18

# From web container, can reach api:
docker exec web curl http://api:3000

# View DNS config
docker exec web cat /etc/resolv.conf
```

### Network Isolation

```bash
# Containers on different networks are isolated
docker network create frontend
docker network create backend

# Web server on frontend only
docker run -d --name web --network frontend nginx

# Database on backend only
docker run -d --name db --network backend postgres

# API on both networks (bridge between frontend and backend)
docker run -d --name api --network frontend myapi
docker network connect backend api
```

## Volumes and Storage

### Volume Types

#### Named Volumes (Recommended)
```bash
# Create named volume
docker volume create mydata

# Use in container
docker run -d -v mydata:/data postgres

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Volume location on host
docker volume inspect mydata | grep Mountpoint
# Usually: /var/lib/docker/volumes/mydata/_data

# Remove volume
docker volume rm mydata

# Remove unused volumes
docker volume prune
```

#### Bind Mounts
```bash
# Mount host directory (absolute path)
docker run -d -v /host/path:/container/path nginx

# Mount current directory
docker run -d -v $(pwd):/app node:18

# Read-only mount
docker run -d -v $(pwd):/app:ro nginx

# Mount with specific permissions
docker run -d -v $(pwd):/app:rw nginx
```

#### tmpfs Mounts (Memory)
```bash
# Store in memory (not persisted)
docker run -d --tmpfs /tmp nginx

# With size limit
docker run -d --tmpfs /tmp:size=100m nginx

# Useful for temporary files, sensitive data
```

### Volume Drivers

```bash
# Create volume with specific driver
docker volume create --driver local myvolume

# Create with options
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.1,rw \
  --opt device=:/path/to/dir \
  nfs-volume
```

### Advanced Volume Usage

#### Backup and Restore
```bash
# Backup volume to tar file
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz -C /data .

# Restore from backup
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/backup.tar.gz -C /data

# Copy between volumes
docker run --rm \
  -v source-vol:/source \
  -v target-vol:/target \
  alpine sh -c "cp -av /source/. /target/"
```

#### Volume Permissions
```bash
# Run as specific user
docker run -d \
  --user 1000:1000 \
  -v mydata:/data \
  myapp

# Initialize volume with permissions
docker run --rm \
  -v mydata:/data \
  alpine chown -R 1000:1000 /data
```

#### Shared Volumes
```bash
# Multiple containers sharing same volume
docker volume create shared-data

# Container 1 writes
docker run -d --name writer -v shared-data:/data alpine \
  sh -c 'while true; do echo $(date) >> /data/log.txt; sleep 1; done'

# Container 2 reads
docker run -d --name reader -v shared-data:/data alpine \
  sh -c 'tail -f /data/log.txt'
```

### Volume Management Best Practices

```bash
# List volumes with filter
docker volume ls --filter dangling=true

# Inspect volume usage
docker system df -v

# Remove specific volume
docker volume rm myvolume

# Remove all unused volumes
docker volume prune -f

# Create volume with labels
docker volume create --label project=myapp myvolume

# Filter by label
docker volume ls --filter label=project=myapp
```

## Multi-Container Applications

### Manual Multi-Container Setup

```bash
# Create network
docker network create myapp-network

# Start database
docker run -d \
  --name postgres-db \
  --network myapp-network \
  -v postgres-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  postgres:15

# Start Redis cache
docker run -d \
  --name redis-cache \
  --network myapp-network \
  -v redis-data:/data \
  redis:7-alpine

# Start backend API
docker run -d \
  --name backend-api \
  --network myapp-network \
  -p 4000:4000 \
  -e DATABASE_URL=postgresql://postgres:secret@postgres-db:5432/myapp \
  -e REDIS_URL=redis://redis-cache:6379 \
  mybackend:latest

# Start frontend
docker run -d \
  --name frontend \
  --network myapp-network \
  -p 3000:3000 \
  -e API_URL=http://backend-api:4000 \
  myfrontend:latest
```

## Docker Compose Basics

### What is Docker Compose?
Tool for defining and running multi-container Docker applications using YAML files.

> **Modern Note**: Docker Compose v2 is now integrated into the Docker CLI as `docker compose` (not `docker-compose`). The `version` field in YAML files is optional in v2+.

### Installation
```bash
# Docker Compose v2 is included with Docker Desktop
# For Linux, install Docker Compose plugin:
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Or install standalone (older method)
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker compose version
```

### Basic docker-compose.yml

> **Note**: The `version` field is optional in Docker Compose v2+. It's included here for compatibility with older versions.

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend

  api:
    image: node:18-alpine
    working_dir: /app
    volumes:
      - ./api:/app
    command: node server.js
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    networks:
      - frontend
      - backend
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  postgres-data:
```

### Docker Compose Commands

```bash
# Start all services
docker compose up

# Start in detached mode
docker compose up -d

# Build images and start
docker compose up --build

# Start specific service
docker compose up web

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v

# View running services
docker compose ps

# View logs
docker compose logs

# Follow logs
docker compose logs -f

# Logs for specific service
docker compose logs -f web

# Execute command in service
docker compose exec web sh

# Build/rebuild services
docker compose build

# Pull latest images
docker compose pull

# Restart services
docker compose restart

# Pause services
docker compose pause

# Unpause services
docker compose unpause

# Stop services (don't remove)
docker compose stop

# Start stopped services
docker compose start

# View configuration
docker compose config

# Validate compose file
docker compose config --quiet
```

### Docker Compose Service Configuration

#### Build Configuration
```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        - NODE_VERSION=18
        - BUILD_DATE=${BUILD_DATE}
    image: myapp:latest
    container_name: myapp-container
```

#### Ports
```yaml
services:
  web:
    ports:
      - "8080:80"           # host:container
      - "8443:443"
      - "127.0.0.1:9000:9000"  # specific IP
    expose:
      - "3000"              # expose to other services only
```

#### Volumes
```yaml
services:
  app:
    volumes:
      - ./src:/app/src                    # bind mount
      - node_modules:/app/node_modules    # named volume
      - /app/logs                          # anonymous volume
      - type: bind
        source: ./config
        target: /app/config
        read_only: true
```

#### Environment Variables
```yaml
services:
  app:
    environment:
      NODE_ENV: production
      API_KEY: ${API_KEY}          # from .env file
      DATABASE_URL: "postgresql://user:pass@db:5432/mydb"
    env_file:
      - .env
      - .env.production
```

#### Dependencies and Health Checks
```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

#### Resource Limits
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

#### Restart Policies
```yaml
services:
  app:
    restart: always              # always, unless-stopped, on-failure, no
```

### Complete Docker Compose Examples

#### Web Application Stack
```yaml
version: '3.8'

services:
  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - frontend
      - api
    networks:
      - frontend
    restart: unless-stopped

  # React frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    environment:
      - REACT_APP_API_URL=http://localhost/api
    networks:
      - frontend
    restart: unless-stopped

  # Node.js API
  api:
    build: ./api
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:${DB_PASSWORD}@postgres:5432/${DB_NAME}
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - frontend
      - backend
    restart: unless-stopped
    volumes:
      - ./api/logs:/app/logs

  # PostgreSQL database
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis cache
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend
    restart: unless-stopped

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  postgres-data:
  redis-data:
```

#### Development Environment
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    volumes:
      - ./src:/app/src
      - /app/node_modules
    ports:
      - "3000:3000"
      - "9229:9229"  # debugger
    environment:
      - NODE_ENV=development
      - DEBUG=app:*
    command: npm run dev
    stdin_open: true
    tty: true

  db:
    image: postgres:15-alpine
    ports:
      - "5432:5432"  # expose for local tools
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: devdb
    volumes:
      - postgres-dev-data:/var/lib/postgresql/data

volumes:
  postgres-dev-data:
```

### Environment Variables with .env

**.env file:**
```bash
# .env
DB_PASSWORD=supersecret
DB_NAME=myapp
API_KEY=abc123xyz
NODE_ENV=production
```

**Use in compose:**
```yaml
services:
  app:
    environment:
      - DATABASE_PASSWORD=${DB_PASSWORD}
      - DATABASE_NAME=${DB_NAME}
      - API_KEY=${API_KEY}
```

### Docker Compose Profiles

```yaml
version: '3.8'

services:
  app:
    image: myapp
    # Always runs

  db:
    image: postgres
    # Always runs

  debug-tools:
    image: debug-container
    profiles: ["debug"]
    # Only runs with: docker compose --profile debug up

  testing:
    image: test-runner
    profiles: ["test"]
    # Only runs with: docker compose --profile test up
```

```bash
# Run with profile
docker compose --profile debug up
docker compose --profile test run testing
```

## Building Docker Images

### Build Context

```bash
# Build from current directory
docker build -t myapp:1.0 .

# Build from different directory
docker build -t myapp:1.0 ./app

# Build from Git repository
docker build -t myapp:1.0 https://github.com/user/repo.git

# Build with specific Dockerfile
docker build -t myapp:1.0 -f Dockerfile.prod .

# Build with build arguments
docker build -t myapp:1.0 --build-arg VERSION=1.0 .
```

### Efficient Image Building

#### Layer Caching
```dockerfile
# Optimize layer caching - put changing files last
FROM node:18-alpine

WORKDIR /app

# These change less frequently - cached
COPY package*.json ./
RUN npm ci --only=production

# Source code changes frequently - separate layer
COPY . .

CMD ["node", "server.js"]
```

#### Minimize Layers
```dockerfile
# ❌ Bad - each RUN creates a layer
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean

# ✅ Good - single layer
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Image Tags and Versioning

```bash
# Tag with version
docker build -t myapp:1.0.0 .

# Multiple tags
docker build -t myapp:1.0.0 -t myapp:1.0 -t myapp:latest .

# Tag existing image
docker tag myapp:1.0.0 myapp:latest
docker tag myapp:1.0.0 myregistry.com/myapp:1.0.0

# Remove tag
docker rmi myapp:1.0.0
```

### BuildKit Features

```bash
# Enable BuildKit (better performance)
export DOCKER_BUILDKIT=1

# Or per-build
DOCKER_BUILDKIT=1 docker build -t myapp .

# Show detailed build output
docker build --progress=plain -t myapp .

# Build with secrets (not stored in image)
docker build --secret id=aws,src=$HOME/.aws/credentials -t myapp .
```

**Dockerfile with secrets:**
```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=aws \
    AWS_CREDENTIALS=$(cat /run/secrets/aws) && \
    echo "Using credentials without storing them"
```

## Docker Registries

### Docker Hub

#### Push to Docker Hub
```bash
# Login
docker login

# Tag for Docker Hub (username/image:tag)
docker tag myapp:1.0 username/myapp:1.0

# Push
docker push username/myapp:1.0

# Pull
docker pull username/myapp:1.0

# Logout
docker logout
```

#### Docker Hub Organizations
```bash
# Tag for organization
docker tag myapp:1.0 organization/myapp:1.0

# Push to organization
docker push organization/myapp:1.0
```

### Private Registries

#### Run Local Registry
```bash
# Start registry container
docker run -d -p 5000:5000 --name registry registry:2

# Tag for local registry
docker tag myapp:1.0 localhost:5000/myapp:1.0

# Push to local registry
docker push localhost:5000/myapp:1.0

# Pull from local registry
docker pull localhost:5000/myapp:1.0
```

#### Secure Registry with Authentication
```bash
# Create password file
mkdir -p auth
docker run --rm --entrypoint htpasswd \
  registry:2 -Bbn username password > auth/htpasswd

# Run registry with auth
docker run -d \
  -p 5000:5000 \
  --name secure-registry \
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2

# Login to secure registry
docker login localhost:5000
```

#### Using Cloud Registries

**AWS ECR:**
```bash
# Login to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# Tag for ECR
docker tag myapp:1.0 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0

# Push
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
```

**Google Container Registry (GCR):**
```bash
# Configure Docker auth
gcloud auth configure-docker

# Tag for GCR
docker tag myapp:1.0 gcr.io/project-id/myapp:1.0

# Push
docker push gcr.io/project-id/myapp:1.0
```

**Azure Container Registry (ACR):**
```bash
# Login to ACR
az acr login --name myregistry

# Tag for ACR
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0

# Push
docker push myregistry.azurecr.io/myapp:1.0
```

### Registry Management

```bash
# List images in Docker Hub repository
curl https://registry.hub.docker.com/v2/repositories/username/myapp/tags

# List images in local registry
curl http://localhost:5000/v2/_catalog

# List tags
curl http://localhost:5000/v2/myapp/tags/list

# Delete image from registry
curl -X DELETE http://localhost:5000/v2/myapp/manifests/sha256:abc123
```

## Do's and Don'ts

### ✅ DO:

1. **Use Docker Networks for Container Communication**
```bash
docker network create myapp-net
docker run --network myapp-net --name db postgres
docker run --network myapp-net --name api myapi
```

2. **Use Named Volumes for Persistent Data**
```bash
docker volume create mydata
docker run -v mydata:/data postgres
```

3. **Use Docker Compose for Multi-Container Apps**
```yaml
# docker-compose.yml - much easier than manual setup
version: '3.8'
services:
  web:
    build: .
    ports: ["3000:3000"]
  db:
    image: postgres
```

4. **Use Health Checks**
```yaml
services:
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
```

5. **Set Resource Limits**
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
```

6. **Use .env Files for Secrets**
```bash
# Never commit secrets to git
# Use .env file and add to .gitignore
echo "DB_PASSWORD=secret" > .env
```

7. **Tag Images with Versions**
```bash
docker build -t myapp:1.0.0 -t myapp:1.0 -t myapp:latest .
```

8. **Use BuildKit for Faster Builds**
```bash
export DOCKER_BUILDKIT=1
docker build -t myapp .
```

### ❌ DON'T:

1. **Don't Use Default Bridge Network for Applications**
```bash
# ❌ Bad - use default bridge
docker run --name app1 nginx
docker run --name app2 api

# ✅ Good - use custom network with DNS
docker network create mynet
docker run --network mynet --name app1 nginx
docker run --network mynet --name app2 api
```

2. **Don't Hardcode Host Paths**
```yaml
# ❌ Bad
volumes:
  - /home/user/data:/data

# ✅ Good
volumes:
  - mydata:/data
```

3. **Don't Expose Database Ports Publicly**
```yaml
# ❌ Bad
services:
  postgres:
    ports:
      - "5432:5432"  # exposed to internet!

# ✅ Good
services:
  postgres:
    expose:
      - "5432"  # only accessible to other services
```

4. **Don't Forget to Clean Up**
```bash
# Regular cleanup
docker system prune
docker volume prune
```

5. **Don't Put Secrets in Images**
```dockerfile
# ❌ Bad
ENV API_KEY=secret123

# ✅ Good - pass at runtime
# docker run -e API_KEY=secret123 myapp
```

6. **Don't Use Host Network in Production**
```bash
# ❌ Bad - no isolation
docker run --network host nginx

# ✅ Good - use bridge network
docker run --network mynet nginx
```

7. **Don't Skip depends_on with Databases**
```yaml
# ❌ Bad - app might start before db
services:
  app:
    image: myapp
  db:
    image: postgres

# ✅ Good
services:
  app:
    depends_on:
      db:
        condition: service_healthy
  db:
    healthcheck:
      test: ["CMD", "pg_isready"]
```

8. **Don't Commit .env Files**
```bash
# Add to .gitignore
echo ".env" >> .gitignore
echo ".env.*" >> .gitignore

# Provide example instead
cp .env .env.example
```

## Troubleshooting Common Issues

### Container Can't Connect to Other Container
```bash
# Check if on same network
docker network inspect mynetwork

# Verify container is running
docker ps

# Test connectivity
docker exec container1 ping container2
docker exec container1 nslookup container2
```

### Volume Data Not Persisting
```bash
# Check volume exists
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Check mount point in container
docker inspect container-name | grep Mounts -A 20
```

### Compose File Not Working
```bash
# Validate syntax
docker compose config

# View parsed configuration
docker compose config

# Check logs for errors
docker compose logs
```

### Image Build Failing
```bash
# Build with full output
docker build --progress=plain --no-cache -t myapp .

# Check build context size
docker build -t myapp . 2>&1 | grep "Sending build context"

# Verify .dockerignore
cat .dockerignore
```

## Quick Command Reference

```bash
# Networking
docker network create <name>        # Create network
docker network ls                   # List networks
docker network inspect <name>       # Inspect network
docker network connect <net> <c>    # Connect container to network

# Volumes
docker volume create <name>         # Create volume
docker volume ls                    # List volumes
docker volume inspect <name>        # Inspect volume
docker volume rm <name>             # Remove volume

# Compose
docker compose up -d                # Start services
docker compose down                 # Stop services
docker compose logs -f              # View logs
docker compose ps                   # List services
docker compose exec <svc> sh        # Execute in service

# Build
docker build -t <name>:<tag> .      # Build image
docker tag <src> <dst>              # Tag image
docker push <name>:<tag>            # Push to registry
docker pull <name>:<tag>            # Pull from registry
```
