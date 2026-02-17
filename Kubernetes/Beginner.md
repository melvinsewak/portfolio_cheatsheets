# Kubernetes Beginner Cheatsheet

## Kubernetes Architecture Overview

### Control Plane Components

**API Server (kube-apiserver)**
- Front-end for Kubernetes control plane
- Exposes Kubernetes API
- All components communicate through it

**etcd**
- Consistent and highly-available key-value store
- Stores all cluster data
- Backup etcd = backup your cluster

**Scheduler (kube-scheduler)**
- Watches for newly created Pods
- Assigns Pods to nodes based on resources
- Considers constraints and availability

**Controller Manager (kube-controller-manager)**
- Runs controller processes
- Node Controller: Monitors node health
- Replication Controller: Maintains correct number of Pods
- Endpoints Controller: Populates Endpoints objects

### Worker Node Components

**kubelet**
- Agent running on each node
- Ensures containers are running in Pods
- Reports node and Pod status to API server

**kube-proxy**
- Network proxy on each node
- Maintains network rules
- Enables communication to Pods

**Container Runtime**
- Software for running containers
- Examples: containerd, CRI-O, Docker

## kubectl Basics

### Configuration and Context

```bash
# View kubectl configuration
kubectl config view

# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context minikube

# Set default namespace
kubectl config set-context --current --namespace=development

# View cluster information
kubectl cluster-info

# View cluster details
kubectl cluster-info dump
```

### Basic Commands

```bash
# Get resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes
kubectl get all

# Get with more details
kubectl get pods -o wide
kubectl get pods --all-namespaces
kubectl get pods -n kube-system

# Describe resources (detailed info)
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe node <node-name>

# Create resources
kubectl create deployment nginx --image=nginx
kubectl create namespace dev

# Apply configuration from file
kubectl apply -f pod.yaml
kubectl apply -f deployment.yaml

# Delete resources
kubectl delete pod <pod-name>
kubectl delete deployment <deployment-name>
kubectl delete -f deployment.yaml

# Get logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs -f <pod-name>  # follow logs
kubectl logs --tail=50 <pod-name>

# Execute commands in Pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -- ls /app
kubectl exec <pod-name> -c <container-name> -- env

# Port forwarding
kubectl port-forward <pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80

# Copy files
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

### Namespace Operations

```bash
# List namespaces
kubectl get namespaces
kubectl get ns

# Create namespace
kubectl create namespace development
kubectl create namespace production

# Set default namespace for context
kubectl config set-context --current --namespace=development

# Get resources in specific namespace
kubectl get pods -n development
kubectl get all -n development

# Get resources in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Delete namespace (deletes all resources in it)
kubectl delete namespace development
```

## Understanding Pods

### What is a Pod?

A Pod is the smallest deployable unit in Kubernetes that represents a single instance of a running process. Pods can contain one or more containers that share:
- Network namespace (same IP address)
- Storage volumes
- Lifecycle

### Simple Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: development
spec:
  containers:
  - name: nginx
    image: nginx:1.25.3
    ports:
    - containerPort: 80
```

### Creating and Managing Pods

```bash
# Create Pod from YAML
kubectl apply -f pod.yaml

# Create Pod imperatively
kubectl run nginx --image=nginx:1.25.3

# List Pods
kubectl get pods
kubectl get pods -o wide

# Get Pod details
kubectl describe pod nginx-pod

# View Pod logs
kubectl logs nginx-pod

# Stream logs
kubectl logs -f nginx-pod

# Execute commands in Pod
kubectl exec nginx-pod -- ls /usr/share/nginx/html
kubectl exec -it nginx-pod -- /bin/bash

# Delete Pod
kubectl delete pod nginx-pod
```

### Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  
  - name: sidecar
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date) > /data/index.html; sleep 10; done"]
    volumeMounts:
    - name: shared-data
      mountPath: /data
  
  volumes:
  - name: shared-data
    emptyDir: {}
```

### Pod with Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: myapp
    image: busybox
    command: ["sleep", "3600"]
    env:
    - name: DATABASE_HOST
      value: "postgres.default.svc.cluster.local"
    - name: DATABASE_PORT
      value: "5432"
    - name: APP_ENV
      value: "production"
```

### Pod with Resource Requests and Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

## Deployments

### What is a Deployment?

A Deployment manages stateless applications by:
- Creating and managing ReplicaSets
- Maintaining desired number of Pod replicas
- Enabling rolling updates and rollbacks
- Self-healing (replacing failed Pods)

### Basic Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.3
        ports:
        - containerPort: 80
```

### Creating Deployments

```bash
# Create from YAML
kubectl apply -f deployment.yaml

# Create imperatively
kubectl create deployment nginx --image=nginx:1.25.3

# Create with replicas
kubectl create deployment nginx --image=nginx:1.25.3 --replicas=3

# Create and expose in one command
kubectl create deployment nginx --image=nginx:1.25.3
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

### Managing Deployments

```bash
# List deployments
kubectl get deployments
kubectl get deploy

# Get deployment details
kubectl describe deployment nginx-deployment

# Get ReplicaSets (created by Deployment)
kubectl get replicasets
kubectl get rs

# View deployment status
kubectl rollout status deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Autoscale deployment
kubectl autoscale deployment nginx-deployment --min=3 --max=10 --cpu-percent=80

# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.26

# Edit deployment
kubectl edit deployment nginx-deployment

# Delete deployment
kubectl delete deployment nginx-deployment
```

### Deployment with Multiple Containers

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: myapp:1.0
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
      
      - name: logging-sidecar
        image: fluent/fluent-bit:latest
        volumeMounts:
        - name: logs
          mountPath: /var/log
      
      volumes:
      - name: logs
        emptyDir: {}
```

### Rolling Updates and Rollbacks

```bash
# Update deployment image (triggers rolling update)
kubectl set image deployment/nginx-deployment nginx=nginx:1.26

# Watch rollout progress
kubectl rollout status deployment/nginx-deployment

# Pause rollout
kubectl rollout pause deployment/nginx-deployment

# Resume rollout
kubectl rollout resume deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment

# View specific revision
kubectl rollout history deployment/nginx-deployment --revision=2

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Restart deployment (rolling restart)
kubectl rollout restart deployment/nginx-deployment
```

## Services

### What is a Service?

A Service provides a stable network endpoint to access Pods. Services enable:
- Load balancing across Pod replicas
- Service discovery (DNS)
- Stable IP address (Pods are ephemeral)
- External access to applications

### Service Types

**ClusterIP (Default)**
- Internal-only access
- Accessible within cluster
- Gets cluster-internal IP

**NodePort**
- Exposes service on each node's IP at a static port
- Accessible from outside cluster via `<NodeIP>:<NodePort>`
- Port range: 30000-32767

**LoadBalancer**
- Creates external load balancer (cloud provider)
- Assigns external IP
- Commonly used in cloud environments

**ExternalName**
- Maps service to external DNS name
- No proxying, just DNS CNAME record

### ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: ClusterIP
  selector:
    app: webapp
  ports:
  - protocol: TCP
    port: 80          # Service port
    targetPort: 3000  # Container port
```

```bash
# Create ClusterIP service
kubectl apply -f service.yaml

# Create imperatively
kubectl expose deployment webapp --port=80 --target-port=3000

# Access service from within cluster
kubectl run test-pod --image=busybox -it --rm -- wget -O- http://webapp-service
```

### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-nodeport
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
    nodePort: 30080  # Optional: specify port (30000-32767)
```

```bash
# Create NodePort service
kubectl apply -f nodeport-service.yaml

# Create imperatively
kubectl expose deployment webapp --type=NodePort --port=80

# Get service details (note the NodePort)
kubectl get service webapp-nodeport

# Access service (minikube)
minikube service webapp-nodeport --url
```

### LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
```

```bash
# Create LoadBalancer service
kubectl apply -f loadbalancer-service.yaml

# Create imperatively
kubectl expose deployment webapp --type=LoadBalancer --port=80

# Get external IP (may show <pending> on minikube)
kubectl get service webapp-loadbalancer

# On minikube, use tunnel to get external IP
minikube tunnel
```

### Service with Multiple Ports

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-multiport
spec:
  type: ClusterIP
  selector:
    app: webapp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
  - name: https
    protocol: TCP
    port: 443
    targetPort: 8443
  - name: metrics
    protocol: TCP
    port: 9090
    targetPort: 9090
```

### Service Discovery

```bash
# Services are accessible via DNS
# Format: <service-name>.<namespace>.svc.cluster.local

# From any Pod in the same namespace
curl http://webapp-service

# From Pod in different namespace
curl http://webapp-service.default.svc.cluster.local

# Environment variables (automatic)
# WEBAPP_SERVICE_HOST
# WEBAPP_SERVICE_PORT

# Test DNS resolution
kubectl run test --image=busybox -it --rm -- nslookup webapp-service
```

### Managing Services

```bash
# List services
kubectl get services
kubectl get svc

# Get service details
kubectl describe service webapp-service

# Get endpoints (Pods backing the service)
kubectl get endpoints webapp-service

# Edit service
kubectl edit service webapp-service

# Delete service
kubectl delete service webapp-service
```

## Namespaces

### What are Namespaces?

Namespaces provide a mechanism for isolating groups of resources within a cluster. Use cases:
- Environment separation (dev, staging, prod)
- Team or project isolation
- Resource quotas per namespace
- Access control boundaries

### Default Namespaces

```bash
# default - Default namespace for resources
# kube-system - Kubernetes system components
# kube-public - Public resources, readable by all
# kube-node-lease - Node heartbeat information
```

### Creating Namespaces

**YAML:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    environment: dev
    team: backend
```

**Imperatively:**
```bash
# Create namespace
kubectl create namespace development
kubectl create namespace staging
kubectl create namespace production

# Create from YAML
kubectl apply -f namespace.yaml
```

### Working with Namespaces

```bash
# List namespaces
kubectl get namespaces
kubectl get ns

# Describe namespace
kubectl describe namespace development

# Create resources in namespace
kubectl apply -f deployment.yaml -n development
kubectl create deployment nginx --image=nginx -n development

# Get resources in namespace
kubectl get pods -n development
kubectl get all -n development

# Get resources in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Set default namespace for current context
kubectl config set-context --current --namespace=development

# Verify current namespace
kubectl config view --minify | grep namespace:

# Delete namespace (deletes all resources)
kubectl delete namespace development
```

### Resource with Namespace

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: development  # Specify namespace
  labels:
    app: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:alpine
        ports:
        - containerPort: 80
```

## Labels and Selectors

### What are Labels?

Labels are key-value pairs attached to objects for identification and organization. They enable:
- Resource grouping and filtering
- Service selectors to find Pods
- ReplicaSet management
- Deployment strategies

### Using Labels

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
  labels:
    app: webapp
    environment: production
    version: v1.0
    tier: frontend
spec:
  containers:
  - name: webapp
    image: nginx
```

### Label Operations

```bash
# Show labels
kubectl get pods --show-labels

# Filter by label
kubectl get pods -l app=webapp
kubectl get pods -l environment=production
kubectl get pods -l 'environment in (production,staging)'
kubectl get pods -l environment!=development

# Add label to existing resource
kubectl label pod webapp-pod tier=frontend

# Update label
kubectl label pod webapp-pod version=v2 --overwrite

# Remove label
kubectl label pod webapp-pod version-

# Label multiple resources
kubectl label pods --all environment=production
```

### Selectors in Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp        # Matches Pods with label app=webapp
    tier: frontend
  ports:
  - port: 80
    targetPort: 8080
```

### Selectors in Deployments

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
      version: v1
  template:
    metadata:
      labels:
        app: webapp
        version: v1
    spec:
      containers:
      - name: webapp
        image: myapp:v1
```

## Basic YAML Manifest Structure

### YAML Fundamentals

```yaml
# Required fields for all Kubernetes objects
apiVersion: v1        # API version (v1, apps/v1, etc.)
kind: Pod            # Type of object
metadata:            # Object metadata
  name: my-pod       # Name (required)
  namespace: default # Namespace (optional, default: default)
  labels:            # Labels for organization
    key: value
  annotations:       # Additional metadata
    key: value
spec:                # Desired state specification
  # Object-specific configuration
```

### Common API Versions

```yaml
# Core resources
apiVersion: v1
kind: Pod, Service, Namespace, ConfigMap, Secret, PersistentVolume, PersistentVolumeClaim

# Apps resources
apiVersion: apps/v1
kind: Deployment, StatefulSet, DaemonSet, ReplicaSet

# Batch resources
apiVersion: batch/v1
kind: Job, CronJob

# Networking
apiVersion: networking.k8s.io/v1
kind: Ingress, NetworkPolicy

# RBAC
apiVersion: rbac.authorization.k8s.io/v1
kind: Role, RoleBinding, ClusterRole, ClusterRoleBinding
```

### Complete Application Example

**deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: production
  labels:
    app: webapp
    version: v1.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
        version: v1.0
    spec:
      containers:
      - name: webapp
        image: myregistry/webapp:1.0
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        - name: ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

**service.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: production
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**Apply both:**
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Or apply all files in directory
kubectl apply -f ./manifests/
```

## Viewing and Debugging

### Get Commands

```bash
# Basic get
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes

# Wide output (more details)
kubectl get pods -o wide

# YAML output
kubectl get pod nginx-pod -o yaml

# JSON output
kubectl get pod nginx-pod -o json

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# Sort by field
kubectl get pods --sort-by=.metadata.creationTimestamp

# Watch for changes
kubectl get pods --watch
kubectl get pods -w
```

### Describe Commands

```bash
# Detailed information about resources
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe service <service-name>
kubectl describe node <node-name>

# Useful for:
# - Events (troubleshooting)
# - Current state vs desired state
# - Resource details and configuration
```

### Logs

```bash
# View logs
kubectl logs <pod-name>

# Multi-container pod (specify container)
kubectl logs <pod-name> -c <container-name>

# Follow logs (stream)
kubectl logs -f <pod-name>

# Last N lines
kubectl logs --tail=50 <pod-name>

# Logs with timestamps
kubectl logs --timestamps <pod-name>

# Previous container logs (if restarted)
kubectl logs <pod-name> --previous

# Logs from all containers in pod
kubectl logs <pod-name> --all-containers=true

# Logs from deployment
kubectl logs deployment/<deployment-name>
```

### Common Debugging Commands

```bash
# Execute command in container
kubectl exec <pod-name> -- ls -la
kubectl exec <pod-name> -- env
kubectl exec <pod-name> -- cat /etc/hosts

# Interactive shell
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- /bin/sh

# Port forward for local testing
kubectl port-forward <pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80

# Copy files to/from pod
kubectl cp /local/file <pod-name>:/container/path
kubectl cp <pod-name>:/container/path /local/file

# Run temporary debug pod
kubectl run debug --image=busybox -it --rm -- /bin/sh

# Check resource usage
kubectl top nodes
kubectl top pods
```

## Do's and Don'ts

### ✅ DO:

1. **Use Deployments, Not Bare Pods**
```yaml
# ✅ Good - use Deployment for self-healing and scaling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:1.25.3
```

2. **Use Namespaces for Organization**
```bash
kubectl create namespace development
kubectl create namespace staging
kubectl create namespace production
```

3. **Set Resource Requests and Limits**
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

4. **Use Labels Consistently**
```yaml
metadata:
  labels:
    app: webapp
    environment: production
    version: v1.0
    tier: frontend
```

5. **Use Specific Image Tags**
```yaml
# ✅ Good
containers:
- name: webapp
  image: nginx:1.25.3
```

6. **Keep YAML in Version Control**
```bash
git add deployment.yaml service.yaml
git commit -m "Add webapp deployment"
```

7. **Use kubectl apply (Declarative)**
```bash
kubectl apply -f deployment.yaml
```

8. **Check Resource Status**
```bash
kubectl get all
kubectl describe deployment webapp
kubectl logs -f <pod-name>
```

### ❌ DON'T:

1. **Don't Deploy Bare Pods in Production**
```yaml
# ❌ Bad - no self-healing, no scaling
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: webapp
    image: nginx
```

2. **Don't Use :latest Tag**
```yaml
# ❌ Bad
containers:
- name: webapp
  image: nginx:latest
```

3. **Don't Skip Resource Limits**
```yaml
# ❌ Bad - can consume all node resources
containers:
- name: webapp
  image: nginx
  # No resources specified
```

4. **Don't Hardcode Configuration**
```yaml
# ❌ Bad
env:
- name: DATABASE_HOST
  value: "192.168.1.100"  # hardcoded IP
```

5. **Don't Ignore Namespaces**
```bash
# ❌ Bad - everything in default namespace
kubectl apply -f deployment.yaml
```

6. **Don't Run as Root**
```yaml
# ❌ Bad - security risk
spec:
  containers:
  - name: webapp
    image: nginx
    # Running as root (default)
```

7. **Don't Use kubectl create for Updates**
```bash
# ❌ Bad - will fail if resource exists
kubectl create -f deployment.yaml

# ✅ Good - creates or updates
kubectl apply -f deployment.yaml
```

8. **Don't Ignore Pod Events**
```bash
# Always check events when debugging
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Common Issues and Solutions

### Pod Not Starting

```bash
# Check Pod status
kubectl get pods
kubectl describe pod <pod-name>

# Common causes:
# - Image pull error: Check image name/tag
# - Insufficient resources: Check resource limits
# - CrashLoopBackOff: Check logs

# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
```

### Service Not Accessible

```bash
# Check service exists
kubectl get service <service-name>

# Check endpoints
kubectl get endpoints <service-name>

# If empty, selector might be wrong
kubectl describe service <service-name>

# Verify Pod labels match service selector
kubectl get pods --show-labels
```

### Cannot Connect to Cluster

```bash
# Check cluster info
kubectl cluster-info

# Check config
kubectl config view

# Check current context
kubectl config current-context

# For minikube
minikube status
minikube start
```

### ImagePullBackOff Error

```bash
# Common causes:
# 1. Wrong image name/tag
# 2. Private registry without imagePullSecrets
# 3. Network issues

# Check events
kubectl describe pod <pod-name>

# Verify image exists
docker pull <image-name>
```

## Quick Command Reference

```bash
# Cluster Info
kubectl cluster-info
kubectl get nodes
kubectl version

# Pods
kubectl get pods
kubectl describe pod <name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- bash
kubectl delete pod <name>

# Deployments
kubectl get deployments
kubectl create deployment <name> --image=<image>
kubectl scale deployment <name> --replicas=3
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>

# Services
kubectl get services
kubectl expose deployment <name> --port=80
kubectl describe service <name>

# Namespaces
kubectl get namespaces
kubectl create namespace <name>
kubectl get pods -n <namespace>
kubectl config set-context --current --namespace=<name>

# Apply/Delete
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>

# Debug
kubectl describe <resource> <name>
kubectl logs -f <pod-name>
kubectl exec -it <pod-name> -- sh
kubectl top nodes
kubectl top pods
```
