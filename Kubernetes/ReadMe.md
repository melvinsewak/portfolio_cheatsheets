# Kubernetes Cheatsheet

Welcome to the Kubernetes cheatsheet! This guide will help you master container orchestration with Kubernetes (k8s).

## Overview

Kubernetes is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications. Originally developed by Google and now maintained by the Cloud Native Computing Foundation (CNCF), Kubernetes has become the industry standard for container orchestration, enabling organizations to run resilient, scalable applications in production.

## Contents

- [Beginner.md](Beginner.md) - Architecture, Pods, Deployments, Services, basic kubectl commands
- [Intermediate.md](Intermediate.md) - ConfigMaps, Secrets, Volumes, Ingress, health checks, resource management
- [Advanced.md](Advanced.md) - StatefulSets, DaemonSets, Jobs, RBAC, Helm, network policies, monitoring

## Quick Reference

### Basic kubectl Commands
```bash
# Get cluster info
kubectl cluster-info

# View nodes
kubectl get nodes

# Create deployment
kubectl create deployment nginx --image=nginx:latest

# Expose deployment as service
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Scale deployment
kubectl scale deployment nginx --replicas=3

# View pods
kubectl get pods

# Delete deployment
kubectl delete deployment nginx
```

### Simple Pod YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Basic Deployment Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: webapp
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
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Service Example
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

## Key Features

- **Container Orchestration**: Automate deployment, scaling, and operations of containerized applications
- **Self-Healing**: Automatic restarts, replacements, and rescheduling of failed containers
- **Horizontal Scaling**: Scale applications up or down automatically based on demand
- **Service Discovery & Load Balancing**: Automatic DNS and load balancing for services
- **Automated Rollouts and Rollbacks**: Gradually deploy changes with zero downtime
- **Secret and Configuration Management**: Securely manage sensitive information
- **Storage Orchestration**: Automatically mount storage systems (local, cloud, network)
- **Batch Execution**: Run batch and CI workloads with Jobs and CronJobs

## Getting Started

### Install kubectl

**Linux:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**macOS:**
```bash
brew install kubectl
```

**Windows:**
```powershell
choco install kubernetes-cli
```

### Verify Installation
```bash
kubectl version --client
```

### Install Minikube (Local Development)

**Linux:**
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**macOS:**
```bash
brew install minikube
```

**Windows:**
```powershell
choco install minikube
```

### Start Minikube
```bash
# Start cluster
minikube start

# Check status
minikube status

# Access dashboard
minikube dashboard

# Stop cluster
minikube stop

# Delete cluster
minikube delete
```

### First Deployment
```bash
# Create deployment
kubectl create deployment hello-k8s --image=nginx

# Expose as service
kubectl expose deployment hello-k8s --type=NodePort --port=80

# Get service URL (minikube)
minikube service hello-k8s --url

# Scale deployment
kubectl scale deployment hello-k8s --replicas=3

# Check status
kubectl get all
```

## Kubernetes Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Control Plane                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  API Server  │  │   Scheduler  │  │  Controller  │  │
│  │              │  │              │  │   Manager    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │               etcd (Key-Value Store)              │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼──────┐  ┌───────▼──────┐  ┌───────▼──────┐
│   Worker 1   │  │   Worker 2   │  │   Worker 3   │
│              │  │              │  │              │
│  ┌────────┐  │  │  ┌────────┐  │  │  ┌────────┐  │
│  │ kubelet│  │  │  │ kubelet│  │  │  │ kubelet│  │
│  └────────┘  │  │  └────────┘  │  │  └────────┘  │
│  ┌────────┐  │  │  ┌────────┐  │  │  ┌────────┐  │
│  │kube-   │  │  │  │kube-   │  │  │  │kube-   │  │
│  │proxy   │  │  │  │proxy   │  │  │  │proxy   │  │
│  └────────┘  │  │  └────────┘  │  │  └────────┘  │
│              │  │              │  │              │
│  Pods:       │  │  Pods:       │  │  Pods:       │
│  ┌──┐ ┌──┐  │  │  ┌──┐ ┌──┐  │  │  ┌──┐ ┌──┐  │
│  │  │ │  │  │  │  │  │ │  │  │  │  │  │ │  │  │
│  └──┘ └──┘  │  │  └──┘ └──┘  │  │  └──┘ └──┘  │
└──────────────┘  └──────────────┘  └──────────────┘
```

## Essential Concepts

- **Pod**: Smallest deployable unit, contains one or more containers
- **Deployment**: Manages stateless application replicas with rolling updates
- **Service**: Stable network endpoint to access Pods
- **Namespace**: Virtual cluster for resource isolation
- **ConfigMap**: Store non-sensitive configuration data
- **Secret**: Store sensitive information (passwords, tokens, keys)
- **Volume**: Persistent or temporary storage for Pods
- **Ingress**: HTTP/HTTPS routing to services
- **Node**: Physical or virtual machine running Kubernetes

## Common Workflows

### Development Workflow
```bash
# 1. Create YAML manifests
# deployment.yaml, service.yaml, etc.

# 2. Apply configuration
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# 3. Verify deployment
kubectl get deployments
kubectl get pods
kubectl get services

# 4. View logs
kubectl logs -f <pod-name>

# 5. Test and iterate
kubectl delete -f deployment.yaml
# Make changes, then reapply
kubectl apply -f deployment.yaml
```

### Production Workflow
```bash
# 1. Apply configurations with namespaces
kubectl create namespace production
kubectl apply -f manifests/ -n production

# 2. Monitor rollout
kubectl rollout status deployment/myapp -n production

# 3. Verify services
kubectl get all -n production

# 4. Check logs and metrics
kubectl logs -f deployment/myapp -n production
kubectl top nodes
kubectl top pods -n production

# 5. Update with zero downtime
kubectl set image deployment/myapp app=myapp:v2 -n production
kubectl rollout status deployment/myapp -n production

# 6. Rollback if needed
kubectl rollout undo deployment/myapp -n production
```

## Quick Tips

✅ **DO:**
- Use namespaces to organize resources
- Set resource requests and limits
- Use labels for resource organization
- Implement health checks (liveness/readiness probes)
- Use ConfigMaps and Secrets for configuration
- Version your YAML manifests in Git
- Use Deployments instead of bare Pods
- Implement proper logging and monitoring

❌ **DON'T:**
- Run containers as root
- Use :latest tag in production
- Store secrets in YAML files
- Skip resource limits
- Deploy directly to production without testing
- Ignore security contexts
- Use single replicas for critical services
- Hardcode configuration in images

## Additional Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Patterns](https://k8spatterns.io/)
- [CNCF Landscape](https://landscape.cncf.io/)
- [Katacoda Interactive Learning](https://www.katacoda.com/courses/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/) - Free online playground
