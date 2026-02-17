# Docker Advanced Cheatsheet

## Advanced Dockerfile Techniques

### Multi-Stage Builds

Multi-stage builds allow you to create smaller production images by separating build and runtime environments.

#### Basic Multi-Stage Build
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

#### Advanced Multi-Stage Build
```dockerfile
# Base stage with common dependencies
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Development stage
FROM base AS development
RUN npm ci
COPY . .
CMD ["npm", "run", "dev"]

# Build stage
FROM base AS builder
RUN npm ci
COPY . .
RUN npm run build && \
    npm run test && \
    npm prune --production

# Production stage
FROM node:18-alpine AS production
WORKDIR /app

# Security: Run as non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy built artifacts and production dependencies
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs package*.json ./

USER nodejs
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node healthcheck.js || exit 1

CMD ["node", "dist/server.js"]
```

#### Multi-Stage with Different Languages
```dockerfile
# Build Go binary
FROM golang:1.21 AS go-builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server

# Build React frontend
FROM node:18 AS frontend-builder
WORKDIR /app
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ ./
RUN npm run build

# Final minimal image
FROM alpine:3.18
RUN apk --no-cache add ca-certificates

WORKDIR /app

# Copy Go binary
COPY --from=go-builder /app/server ./server

# Copy React build
COPY --from=frontend-builder /app/build ./public

EXPOSE 8080
CMD ["./server"]
```

### Build Arguments and Targets

```dockerfile
# Multi-target Dockerfile
FROM node:18-alpine AS base
WORKDIR /app
ARG NODE_ENV
ENV NODE_ENV=$NODE_ENV

# Development target
FROM base AS development
RUN npm install -g nodemon
COPY package*.json ./
RUN npm install
CMD ["nodemon", "server.js"]

# Production target
FROM base AS production
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]

# Testing target
FROM base AS testing
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm test
```

**Build specific target:**
```bash
# Build development image
docker build --target development -t myapp:dev .

# Build production image
docker build --target production -t myapp:prod .

# Build with arguments
docker build --build-arg NODE_ENV=production -t myapp:prod .
```

### Optimizing Dockerfile

#### Using BuildKit Cache Mounts
```dockerfile
# syntax=docker/dockerfile:1

FROM node:18-alpine

WORKDIR /app

# Cache npm dependencies
RUN --mount=type=cache,target=/root/.npm \
    npm install -g npm@latest

COPY package*.json ./

RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

COPY . .

CMD ["node", "server.js"]
```

#### Efficient Layer Ordering
```dockerfile
FROM python:3.11-slim

# Install system dependencies (rarely change)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Python dependencies (change occasionally)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code (changes frequently)
COPY . .

CMD ["python", "app.py"]
```

### Advanced COPY Techniques

```dockerfile
# Copy with exclusions using .dockerignore
COPY . .

# Copy specific files
COPY package*.json ./
COPY tsconfig.json ./

# Copy with ownership
COPY --chown=node:node package*.json ./

# Copy from another stage
COPY --from=builder /app/dist ./dist

# Copy from external image
COPY --from=nginx:latest /etc/nginx/nginx.conf /etc/nginx/

# Copy from build context directory
COPY --from=. /local/path /container/path
```

## Docker Compose Advanced

### Advanced Service Configuration

```yaml
version: '3.8'

services:
  web:
    build:
      context: ./web
      dockerfile: Dockerfile.prod
      args:
        - BUILD_VERSION=${VERSION:-latest}
        - BUILD_DATE=${BUILD_DATE}
      cache_from:
        - myapp:latest
      target: production
    image: myapp:${VERSION:-latest}
    container_name: myapp-web
    hostname: web-server
    domainname: myapp.local
    
    # Networking
    networks:
      frontend:
        ipv4_address: 172.20.0.5
        aliases:
          - web-api
      backend:
    ports:
      - "80:80"
      - "443:443"
    expose:
      - "3000"
    
    # Volumes and configs
    volumes:
      - type: bind
        source: ./config
        target: /etc/myapp
        read_only: true
      - type: volume
        source: app-data
        target: /data
      - type: tmpfs
        target: /tmp
        tmpfs:
          size: 1000000000  # 1GB
    
    configs:
      - source: nginx-config
        target: /etc/nginx/nginx.conf
        mode: 0440
    
    secrets:
      - source: api-key
        target: /run/secrets/api_key
        mode: 0400
    
    # Environment
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=info
    env_file:
      - .env
      - .env.production
    
    # Resource limits
    deploy:
      mode: replicated
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      update_config:
        parallelism: 2
        delay: 10s
        failure_action: rollback
      rollback_config:
        parallelism: 1
        delay: 5s
    
    # Dependencies and ordering
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    
    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # Process management
    init: true
    stop_grace_period: 30s
    stop_signal: SIGTERM
    
    # Logging
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "production"
    
    # Security
    user: "1000:1000"
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
      - /run

  db:
    image: postgres:15-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d:ro
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7-alpine
    command: >
      redis-server
      --requirepass ${REDIS_PASSWORD}
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
    volumes:
      - redis-data:/data
    networks:
      - backend

networks:
  frontend:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
  backend:
    driver: bridge
    internal: true

volumes:
  app-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/app-data
  postgres-data:
  redis-data:

configs:
  nginx-config:
    file: ./nginx/nginx.conf

secrets:
  api-key:
    file: ./secrets/api-key.txt
  db_password:
    external: true
```

### Docker Compose with Override Files

**docker-compose.yml (base):**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    environment:
      - NODE_ENV=production
```

**docker-compose.override.yml (development):**
```yaml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - ./src:/app/src
    environment:
      - NODE_ENV=development
      - DEBUG=*
    ports:
      - "9229:9229"  # debugger
```

**docker-compose.prod.yml (production):**
```yaml
version: '3.8'

services:
  app:
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1'
          memory: 1G
```

```bash
# Development (uses override automatically)
docker compose up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up

# Testing
docker compose -f docker-compose.yml -f docker-compose.test.yml up
```

### Extending Services

```yaml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

x-healthcheck: &default-healthcheck
  interval: 30s
  timeout: 10s
  retries: 3

services:
  web:
    image: myapp
    logging: *default-logging
    healthcheck:
      <<: *default-healthcheck
      test: ["CMD", "curl", "-f", "http://localhost/health"]

  api:
    image: myapi
    logging: *default-logging
    healthcheck:
      <<: *default-healthcheck
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
```

## Docker Swarm Basics

### Initialize Swarm

```bash
# Initialize swarm (manager node)
docker swarm init

# Initialize with specific IP
docker swarm init --advertise-addr 192.168.1.100

# Get join token for workers
docker swarm join-token worker

# Get join token for managers
docker swarm join-token manager

# Join swarm as worker
docker swarm join --token <token> 192.168.1.100:2377

# Leave swarm
docker swarm leave

# Force leave (manager)
docker swarm leave --force
```

### Swarm Management

```bash
# List nodes
docker node ls

# Inspect node
docker node inspect node-name

# Promote worker to manager
docker node promote node-name

# Demote manager to worker
docker node demote node-name

# Remove node
docker node rm node-name

# Update node availability
docker node update --availability drain node-name
docker node update --availability active node-name

# Add labels to node
docker node update --label-add type=webserver node-name
```

### Deploy Services

```bash
# Deploy service
docker service create --name web --replicas 3 -p 80:80 nginx

# List services
docker service ls

# Inspect service
docker service inspect web

# View service logs
docker service logs web

# Scale service
docker service scale web=5

# Update service
docker service update --image nginx:latest web

# Remove service
docker service rm web
```

### Stack Deployment

**stack.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        max_attempts: 3
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == webserver

  api:
    image: myapi:latest
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
      placement:
        preferences:
          - spread: node.labels.datacenter

networks:
  overlay:
    driver: overlay

volumes:
  data:
```

```bash
# Deploy stack
docker stack deploy -c stack.yml myapp

# List stacks
docker stack ls

# List stack services
docker stack services myapp

# List stack tasks
docker stack ps myapp

# Remove stack
docker stack rm myapp
```

## Security Best Practices

### Image Security

#### Scan Images for Vulnerabilities
```bash
# Using Docker Scout (built-in)
docker scout cves myapp:latest

# Using Trivy
docker run aquasec/trivy image myapp:latest

# Using Snyk
snyk container test myapp:latest

# Using Clair
clairctl analyze myapp:latest
```

#### Use Minimal Base Images
```dockerfile
# ❌ Avoid full OS images
FROM ubuntu:22.04

# ✅ Use minimal images
FROM alpine:3.18
FROM node:18-alpine
FROM python:3.11-slim
FROM gcr.io/distroless/python3

# ✅ Use distroless for ultimate security
FROM gcr.io/distroless/static-debian11
COPY --from=builder /app/binary /app/binary
CMD ["/app/binary"]
```

#### Use Specific Image Versions
```dockerfile
# ❌ Don't use latest
FROM node:latest

# ✅ Pin specific versions
FROM node:18.17.1-alpine3.18

# ✅ Use SHA for immutability
FROM node@sha256:abc123...
```

### Container Security

#### Run as Non-Root User
```dockerfile
FROM node:18-alpine

# Create user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set ownership
COPY --chown=nodejs:nodejs . /app

# Switch to user
USER nodejs

CMD ["node", "server.js"]
```

#### Use Read-Only Root Filesystem
```bash
# Run with read-only root filesystem
docker run --read-only --tmpfs /tmp myapp

# In Compose
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
```

#### Drop Capabilities
```bash
# Drop all capabilities and add only needed ones
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# In Compose
services:
  app:
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

#### Security Options
```bash
# Enable no-new-privileges
docker run --security-opt=no-new-privileges myapp

# Use AppArmor profile
docker run --security-opt apparmor=docker-default myapp

# Use SELinux
docker run --security-opt label=level:s0:c100,c200 myapp

# In Compose
services:
  app:
    security_opt:
      - no-new-privileges:true
      - apparmor:docker-default
```

### Secrets Management

#### Docker Secrets (Swarm)
```bash
# Create secret from file
docker secret create db_password ./password.txt

# Create secret from stdin
echo "mypassword" | docker secret create db_password -

# List secrets
docker secret ls

# Use in service
docker service create \
  --name mysql \
  --secret db_password \
  -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_password \
  mysql:8.0

# Remove secret
docker secret rm db_password
```

#### Docker Secrets in Compose
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
    # or external: true for swarm secrets
```

#### Environment Variables from Files
```bash
# Store secret in file (not in environment)
echo "supersecret" > /run/secrets/api_key

# Read from file in application
# Instead of: API_KEY=supersecret
# Use: API_KEY_FILE=/run/secrets/api_key
```

### Network Security

```yaml
# Isolate networks
version: '3.8'

services:
  frontend:
    networks:
      - public
      - internal

  backend:
    networks:
      - internal
    # Not exposed to public

  db:
    networks:
      - database
    # Completely isolated

networks:
  public:
    driver: bridge
  internal:
    driver: bridge
    internal: true  # No external access
  database:
    driver: bridge
    internal: true
```

### Content Trust and Signing

```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Pull verified image
docker pull nginx:latest  # Will verify signature

# Push signed image
docker push myapp:latest  # Will sign image

# Generate signing keys
docker trust key generate mykey

# Sign image
docker trust sign myapp:latest

# Verify signature
docker trust inspect myapp:latest
```

## Image Optimization

### Minimize Image Size

#### Use Multi-Stage Builds
```dockerfile
# Build stage (large)
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

# Production stage (small)
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

#### Remove Build Dependencies
```dockerfile
FROM python:3.11-slim

# Install build dependencies, then remove
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    gcc && \
    pip install --no-cache-dir -r requirements.txt && \
    apt-get purge -y --auto-remove \
    build-essential \
    gcc && \
    rm -rf /var/lib/apt/lists/*
```

#### Optimize Layer Caching
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files first (cached if unchanged)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code last (changes frequently)
COPY . .

CMD ["node", "server.js"]
```

### Image Analysis

```bash
# Show image layers and sizes
docker history myapp:latest

# Show detailed layer information
docker history --no-trunc --format "table {{.CreatedBy}}\t{{.Size}}" myapp:latest

# Analyze image with dive
dive myapp:latest

# Check image size
docker images myapp:latest

# Compare image sizes
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Best Practices for Small Images

```dockerfile
# ✅ Good - 50MB
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
CMD ["node", "server.js"]

# vs

# ❌ Bad - 900MB
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]
```

## Performance Optimization

### Build Performance

#### Enable BuildKit
```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Or per-build
DOCKER_BUILDKIT=1 docker build -t myapp .
```

#### Use Build Cache
```bash
# Build with inline cache
docker build --cache-from myapp:latest -t myapp:new .

# Push with cache
docker build --cache-from myapp:latest --cache-to type=inline -t myapp:latest .
```

#### Parallel Builds
```bash
# Build multiple targets in parallel
docker build --target dev -t myapp:dev . &
docker build --target prod -t myapp:prod . &
wait
```

### Runtime Performance

#### Resource Limits
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
```

#### Storage Drivers
```bash
# Check current driver
docker info | grep "Storage Driver"

# Best for most cases: overlay2
# Configure in /etc/docker/daemon.json
{
  "storage-driver": "overlay2"
}
```

#### Logging Drivers
```yaml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### Network Performance

```yaml
# Use host network for maximum performance (no isolation)
services:
  app:
    network_mode: host

# Or optimize bridge network
networks:
  mynetwork:
    driver: bridge
    driver_opts:
      com.docker.network.driver.mtu: 1500
```

## Troubleshooting

### Debugging Containers

#### Interactive Debugging
```bash
# Run with interactive shell
docker run -it myapp sh

# Execute shell in running container
docker exec -it mycontainer sh

# Run as root for debugging
docker exec -it -u root mycontainer sh

# Attach to running container
docker attach mycontainer

# Start stopped container in interactive mode
docker start -ai mycontainer
```

#### Inspect Container State
```bash
# View detailed container info
docker inspect mycontainer

# Check specific values
docker inspect --format='{{.State.Status}}' mycontainer
docker inspect --format='{{.NetworkSettings.IPAddress}}' mycontainer
docker inspect --format='{{json .Config.Env}}' mycontainer | jq

# View container processes
docker top mycontainer

# View resource usage
docker stats mycontainer

# View container events
docker events --filter container=mycontainer

# View container changes
docker diff mycontainer
```

#### Log Analysis
```bash
# View logs
docker logs mycontainer

# Follow logs
docker logs -f mycontainer

# Show last N lines
docker logs --tail 100 mycontainer

# Show logs with timestamps
docker logs -t mycontainer

# Show logs since specific time
docker logs --since 2024-01-01T00:00:00 mycontainer
docker logs --since 60m mycontainer

# Filter logs
docker logs mycontainer 2>&1 | grep ERROR
```

### Debugging Build Issues

```bash
# Build with full output
docker build --progress=plain --no-cache -t myapp .

# Debug specific build stage
docker build --target builder -t myapp:debug .
docker run -it myapp:debug sh

# Check build context size
docker build -t myapp . 2>&1 | head -n 1

# List files in build context
tar -czh -f - . | tar -t | head -20
```

### Network Debugging

```bash
# Inspect network
docker network inspect mynetwork

# Test connectivity
docker exec container1 ping container2
docker exec container1 nslookup container2
docker exec container1 telnet container2 3000

# Install debugging tools
docker exec -it mycontainer sh -c 'apk add --no-cache curl netcat-openbsd'

# Test HTTP endpoints
docker exec mycontainer curl -v http://other-container:3000/health

# Check port bindings
docker port mycontainer

# View network packets (requires tcpdump)
docker run --net=container:mycontainer nicolaka/netshoot tcpdump
```

### Volume Debugging

```bash
# Inspect volume
docker volume inspect myvolume

# Check volume data
docker run --rm -v myvolume:/data alpine ls -la /data

# Backup volume for inspection
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine \
  tar czf /backup/volume-backup.tar.gz -C /data .

# Check permissions
docker run --rm -v myvolume:/data alpine ls -ln /data

# Fix permissions
docker run --rm -v myvolume:/data alpine chown -R 1000:1000 /data
```

### Common Issues

#### Container Exits Immediately
```bash
# Check exit code
docker inspect --format='{{.State.ExitCode}}' mycontainer

# Common exit codes:
# 0 - Success
# 1 - Application error
# 137 - Killed (OOM or manual)
# 139 - Segmentation fault
# 143 - Graceful termination (SIGTERM)

# Check logs for errors
docker logs mycontainer

# Run interactively to debug
docker run -it myapp sh
```

#### Out of Memory
```bash
# Check if OOM killed
docker inspect --format='{{.State.OOMKilled}}' mycontainer

# View memory usage
docker stats mycontainer

# Set memory limit
docker run -m 512m myapp

# Check kernel logs
dmesg | grep -i oom
```

#### Permission Denied
```bash
# Check user
docker exec mycontainer whoami
docker exec mycontainer id

# Check file permissions
docker exec mycontainer ls -la /path

# Run as root temporarily
docker exec -u root mycontainer chown app:app /path

# Fix in Dockerfile
RUN chown -R appuser:appuser /app
USER appuser
```

#### DNS Issues
```bash
# Check DNS resolution
docker exec mycontainer nslookup google.com
docker exec mycontainer cat /etc/resolv.conf

# Use custom DNS
docker run --dns 8.8.8.8 myapp

# Or in daemon.json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

### System-Level Debugging

```bash
# Check Docker daemon status
systemctl status docker

# View daemon logs
journalctl -u docker.service -f

# Check Docker disk usage
docker system df

# Detailed disk usage
docker system df -v

# Clean up
docker system prune -a

# Check Docker info
docker info

# Check Docker version
docker version

# Run Docker in debug mode
dockerd --debug
```

### Performance Issues

```bash
# Identify resource-heavy containers
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Check I/O performance
docker stats --format "table {{.Container}}\t{{.BlockIO}}"

# Identify large images
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -h -k3

# Check layer sizes
docker history myapp:latest --no-trunc

# Profile application
docker run --cap-add=SYS_PTRACE myapp
```

## Do's and Don'ts

### ✅ DO:

1. **Use Multi-Stage Builds**
```dockerfile
FROM node:18 AS builder
RUN npm ci && npm run build

FROM node:18-alpine
COPY --from=builder /app/dist ./dist
```

2. **Scan Images for Vulnerabilities**
```bash
docker scout cves myapp:latest
trivy image myapp:latest
```

3. **Run as Non-Root**
```dockerfile
USER nodejs
```

4. **Use Read-Only Filesystems**
```bash
docker run --read-only --tmpfs /tmp myapp
```

5. **Implement Health Checks**
```dockerfile
HEALTHCHECK CMD curl -f http://localhost/health || exit 1
```

6. **Pin Dependency Versions**
```dockerfile
FROM node:18.17.1-alpine3.18
```

7. **Use .dockerignore**
```
node_modules
.git
*.md
.env
```

8. **Set Resource Limits**
```yaml
deploy:
  resources:
    limits:
      cpus: '1'
      memory: 1G
```

### ❌ DON'T:

1. **Don't Use Latest Tag in Production**
```bash
# ❌ Bad
FROM node:latest

# ✅ Good
FROM node:18.17.1-alpine3.18
```

2. **Don't Run as Root**
```dockerfile
# ❌ Bad - runs as root
CMD ["node", "server.js"]

# ✅ Good
USER node
CMD ["node", "server.js"]
```

3. **Don't Store Secrets in Images**
```dockerfile
# ❌ Bad
ENV API_KEY=secret123

# ✅ Good
# Pass at runtime with --secret or -e
```

4. **Don't Ignore Security Scans**
```bash
# Always scan before deploying
docker scout cves myapp:latest
```

5. **Don't Use Large Base Images**
```dockerfile
# ❌ Bad - 900MB
FROM node:18

# ✅ Good - 50MB
FROM node:18-alpine
```

6. **Don't Skip Health Checks**
```yaml
# Always include health checks
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/health"]
```

7. **Don't Expose Unnecessary Capabilities**
```yaml
cap_drop:
  - ALL
cap_add:
  - NET_BIND_SERVICE
```

8. **Don't Forget to Clean Build Cache**
```bash
docker builder prune
docker system prune -a
```

## Quick Command Reference

```bash
# Multi-stage builds
docker build --target production -t myapp:prod .

# Security scanning
docker scout cves myapp:latest

# Secrets (Swarm)
docker secret create api_key ./key.txt

# Swarm
docker swarm init
docker service create --name web --replicas 3 nginx
docker stack deploy -c stack.yml myapp

# Debugging
docker logs -f --tail 100 mycontainer
docker exec -it mycontainer sh
docker inspect mycontainer
docker stats mycontainer

# Performance
docker system df
docker system prune -a
docker history myapp:latest

# Advanced networking
docker network create --driver overlay mynet
docker network inspect mynet
```

## Additional Resources

- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Docker BuildKit](https://docs.docker.com/build/buildkit/)
- [Container Security Guide](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
