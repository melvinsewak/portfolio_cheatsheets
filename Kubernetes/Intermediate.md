# Kubernetes Intermediate Cheatsheet

## ConfigMaps

### What are ConfigMaps?

ConfigMaps store non-sensitive configuration data as key-value pairs. They decouple configuration from container images, making applications portable across environments.

### Creating ConfigMaps

#### From Literal Values
```bash
# Create ConfigMap from literal values
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=postgres.default.svc.cluster.local \
  --from-literal=DATABASE_PORT=5432 \
  --from-literal=APP_ENV=production

# View ConfigMap
kubectl get configmap app-config
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

#### From File
```bash
# Create from file
kubectl create configmap nginx-config --from-file=nginx.conf

# Create from multiple files
kubectl create configmap app-config \
  --from-file=config.json \
  --from-file=settings.yaml

# Create from directory
kubectl create configmap app-config --from-file=./config/
```

#### From YAML
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  # Simple key-value pairs
  DATABASE_HOST: "postgres.default.svc.cluster.local"
  DATABASE_PORT: "5432"
  APP_ENV: "production"
  LOG_LEVEL: "info"
  
  # Multi-line configuration
  app.properties: |
    server.port=8080
    server.host=0.0.0.0
    logging.level=INFO
    database.pool.size=10
  
  nginx.conf: |
    server {
        listen 80;
        server_name localhost;
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
```

### Using ConfigMaps in Pods

#### As Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: myapp:1.0
    env:
    # Single environment variable from ConfigMap
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_HOST
    
    # All keys as environment variables
    envFrom:
    - configMapRef:
        name: app-config
```

#### As Volume Mount
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf  # Mount specific file
  
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
```

#### Mount All Keys as Files
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: config
      mountPath: /config
      # Each key becomes a file in /config/
  
  volumes:
  - name: config
    configMap:
      name: app-config
```

### ConfigMap in Deployment

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
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: app-config
        volumeMounts:
        - name: config-files
          mountPath: /app/config
      
      volumes:
      - name: config-files
        configMap:
          name: app-config
          items:
          - key: app.properties
            path: application.properties
```

### Managing ConfigMaps

```bash
# List ConfigMaps
kubectl get configmaps
kubectl get cm

# Describe ConfigMap
kubectl describe configmap app-config

# View ConfigMap content
kubectl get configmap app-config -o yaml

# Edit ConfigMap
kubectl edit configmap app-config

# Delete ConfigMap
kubectl delete configmap app-config

# Create/update from YAML
kubectl apply -f configmap.yaml

# Update from file
kubectl create configmap app-config --from-file=config.json --dry-run=client -o yaml | kubectl apply -f -
```

## Secrets

### What are Secrets?

Secrets store sensitive information like passwords, tokens, and keys. They are base64-encoded (not encrypted by default) and should be used with encryption at rest enabled.

### Creating Secrets

#### Generic Secret from Literals
```bash
# Create secret from literals
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=supersecret123

# View secret (data is base64 encoded)
kubectl get secret db-credentials -o yaml
```

#### From Files
```bash
# Create from file
kubectl create secret generic tls-secret \
  --from-file=tls.crt \
  --from-file=tls.key

# Create from env file
kubectl create secret generic app-secrets --from-env-file=secrets.env
```

#### Docker Registry Secret
```bash
# Create docker-registry secret
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.io \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=user@example.com
```

#### TLS Secret
```bash
# Create TLS secret from certificate files
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
```

#### From YAML
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: default
type: Opaque
data:
  # Base64 encoded values
  username: YWRtaW4=        # admin
  password: c3VwZXJzZWNyZXQ= # supersecret
```

```bash
# Encode values
echo -n 'admin' | base64
echo -n 'supersecret' | base64

# Decode values
echo 'YWRtaW4=' | base64 --decode
```

#### String Data (No Base64 Encoding Needed)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  # Plain text - automatically base64 encoded
  username: admin
  password: supersecret123
  database_url: postgresql://admin:supersecret123@postgres:5432/mydb
```

### Using Secrets in Pods

#### As Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: myapp:1.0
    env:
    # Single environment variable from Secret
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
    
    # All keys as environment variables
    envFrom:
    - secretRef:
        name: db-credentials
```

#### As Volume Mount
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: myapp:1.0
    volumeMounts:
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true
  
  volumes:
  - name: secrets
    secret:
      secretName: db-credentials
      # Each key becomes a file
      # /etc/secrets/username
      # /etc/secrets/password
```

#### Mount Specific Secret Keys
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /app/secrets
      readOnly: true
  
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
      items:
      - key: username
        path: db-user.txt
      - key: password
        path: db-pass.txt
```

#### ImagePullSecrets for Private Registry
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  containers:
  - name: app
    image: myregistry.io/private/app:1.0
  imagePullSecrets:
  - name: regcred
```

### Secrets in Deployment

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
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: myapp:1.0
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: database_url
        volumeMounts:
        - name: app-secrets
          mountPath: /run/secrets
          readOnly: true
      
      volumes:
      - name: app-secrets
        secret:
          secretName: app-secrets
      
      imagePullSecrets:
      - name: regcred
```

### Managing Secrets

```bash
# List secrets
kubectl get secrets

# Describe secret (doesn't show values)
kubectl describe secret db-credentials

# View secret (base64 encoded)
kubectl get secret db-credentials -o yaml

# View decoded secret
kubectl get secret db-credentials -o jsonpath='{.data.username}' | base64 --decode
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 --decode

# Edit secret
kubectl edit secret db-credentials

# Delete secret
kubectl delete secret db-credentials

# Create/update from YAML
kubectl apply -f secret.yaml
```

## Persistent Volumes (PV) and Claims (PVC)

### What are Persistent Volumes?

**PersistentVolume (PV)**: Cluster-level storage resource provisioned by administrator or dynamically provisioned.

**PersistentVolumeClaim (PVC)**: Request for storage by a user. Binds to available PV.

### Storage Classes

**StorageClass**: Defines different "classes" of storage with different properties (speed, backup policy, etc.)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "10"
  encrypted: "true"
reclaimPolicy: Retain
allowVolumeExpansion: true
```

### PersistentVolume (Manual Provisioning)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
  namespace: default
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

### Access Modes

- **ReadWriteOnce (RWO)**: Mounted as read-write by single node
- **ReadOnlyMany (ROX)**: Mounted as read-only by many nodes
- **ReadWriteMany (RWX)**: Mounted as read-write by many nodes

### Using PVC in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /usr/share/nginx/html
  
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: app-pvc
```

### Using PVC in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 1  # RWO volumes: use 1 replica or RWX
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
        volumeMounts:
        - name: data
          mountPath: /app/data
      
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: app-pvc
```

### Dynamic Provisioning

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard  # Uses default or specified StorageClass
  resources:
    requests:
      storage: 10Gi
```

```bash
# List storage classes
kubectl get storageclass
kubectl get sc

# Get PersistentVolumes
kubectl get pv

# Get PersistentVolumeClaims
kubectl get pvc

# Describe PVC
kubectl describe pvc app-pvc

# Delete PVC
kubectl delete pvc app-pvc
```

### Volume Types in Pods

#### emptyDir (Temporary)
```yaml
volumes:
- name: cache
  emptyDir: {}
  # Deleted when Pod is removed
```

#### hostPath (Node's filesystem)
```yaml
volumes:
- name: host-storage
  hostPath:
    path: /data
    type: Directory
  # Risky - ties Pod to specific node
```

#### configMap Volume
```yaml
volumes:
- name: config
  configMap:
    name: app-config
```

#### secret Volume
```yaml
volumes:
- name: secrets
  secret:
    secretName: app-secrets
```

## Ingress

### What is Ingress?

Ingress provides HTTP/HTTPS routing to services based on hostnames and paths. It offers:
- Load balancing
- SSL termination
- Name-based virtual hosting
- Path-based routing

### Prerequisites

Requires an Ingress Controller (nginx, traefik, etc.)

```bash
# Install NGINX Ingress Controller (minikube)
minikube addons enable ingress

# Install NGINX Ingress Controller (general)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Verify installation
kubectl get pods -n ingress-nginx
```

### Basic Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service
            port:
              number: 80
```

### Multiple Path Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multipath-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Multiple Host Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multihost-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Ingress with TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret  # TLS certificate secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service
            port:
              number: 80
```

### Default Backend

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-backend-ingress
spec:
  defaultBackend:
    service:
      name: default-service
      port:
        number: 80
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 8080
```

### Ingress Annotations (NGINX)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: annotated-ingress
  annotations:
    # Rewrite rules
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    
    # SSL redirect
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"
    
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    
    # Proxy settings
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    
    # Authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

### Managing Ingress

```bash
# List Ingress resources
kubectl get ingress
kubectl get ing

# Describe Ingress
kubectl describe ingress webapp-ingress

# Get Ingress with addresses
kubectl get ingress -o wide

# Edit Ingress
kubectl edit ingress webapp-ingress

# Delete Ingress
kubectl delete ingress webapp-ingress

# Test Ingress (add to /etc/hosts first)
curl http://myapp.example.com
```

## Labels and Selectors (Advanced)

### Label Selectors

```bash
# Equality-based
kubectl get pods -l app=webapp
kubectl get pods -l environment=production
kubectl get pods -l app=webapp,environment=production

# Inequality
kubectl get pods -l environment!=production

# Set-based
kubectl get pods -l 'environment in (production,staging)'
kubectl get pods -l 'tier notin (frontend,backend)'
kubectl get pods -l 'app,!version'  # Has app label, no version label
```

### Advanced Selectors in YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    # Match expressions (more powerful than matchLabels)
    matchExpressions:
    - key: app
      operator: In
      values:
      - webapp
      - api
    - key: environment
      operator: NotIn
      values:
      - development
    - key: tier
      operator: Exists
  template:
    metadata:
      labels:
        app: webapp
        environment: production
        tier: frontend
    spec:
      containers:
      - name: webapp
        image: myapp:1.0
```

### Node Selectors

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  nodeSelector:
    hardware: gpu
    disktype: ssd
  containers:
  - name: ml-app
    image: ml-app:1.0
```

```bash
# Label node
kubectl label nodes node-1 hardware=gpu
kubectl label nodes node-1 disktype=ssd

# View node labels
kubectl get nodes --show-labels
```

## Rolling Updates and Rollbacks

### Update Strategies

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 5
  strategy:
    type: RollingUpdate  # or Recreate
    rollingUpdate:
      maxUnavailable: 1   # Max pods unavailable during update
      maxSurge: 1         # Max pods above desired count
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
```

### Performing Rolling Update

```bash
# Update image
kubectl set image deployment/webapp webapp=myapp:2.0

# Update with record (saves in rollout history)
kubectl set image deployment/webapp webapp=myapp:2.0 --record

# Watch rollout progress
kubectl rollout status deployment/webapp

# Check rollout status
kubectl rollout status deployment/webapp
# Output: deployment "webapp" successfully rolled out
```

### Rollout History

```bash
# View rollout history
kubectl rollout history deployment/webapp

# View specific revision
kubectl rollout history deployment/webapp --revision=2

# Annotate deployment for better history
kubectl annotate deployment/webapp \
  kubernetes.io/change-cause="Update to version 2.0"
```

### Rollback

```bash
# Rollback to previous version
kubectl rollout undo deployment/webapp

# Rollback to specific revision
kubectl rollout undo deployment/webapp --to-revision=2

# Verify rollback
kubectl rollout status deployment/webapp
```

### Pause and Resume Rollout

```bash
# Pause rollout (for canary deployments)
kubectl rollout pause deployment/webapp

# Make multiple changes
kubectl set image deployment/webapp webapp=myapp:2.1
kubectl set resources deployment/webapp -c webapp --limits=cpu=500m,memory=512Mi

# Resume rollout
kubectl rollout resume deployment/webapp

# Restart deployment (rolling restart)
kubectl rollout restart deployment/webapp
```

## Resource Limits and Requests

### Understanding Resources

**Requests**: Minimum resources guaranteed to container
**Limits**: Maximum resources container can use

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        memory: "128Mi"   # 128 Mebibytes
        cpu: "250m"       # 250 millicores (0.25 CPU)
      limits:
        memory: "256Mi"
        cpu: "500m"
```

### Resource Units

**CPU:**
- 1 CPU = 1000 millicores (m)
- 500m = 0.5 CPU
- 1 = 1 vCPU/core

**Memory:**
- Ki = Kibibyte (1024 bytes)
- Mi = Mebibyte (1024 Ki)
- Gi = Gibibyte (1024 Mi)
- K = Kilobyte (1000 bytes)
- M = Megabyte (1000 K)
- G = Gigabyte (1000 M)

### QoS Classes

Kubernetes assigns QoS classes based on resource settings:

**Guaranteed**: requests = limits for all containers
**Burstable**: requests < limits or only requests set
**BestEffort**: No requests or limits set

```yaml
# Guaranteed QoS
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "256Mi"
    cpu: "500m"

# Burstable QoS
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"

# BestEffort QoS
# No resources specified
```

### ResourceQuota (Namespace Level)

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
    services: "10"
    persistentvolumeclaims: "5"
```

### LimitRange (Default Limits)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
  namespace: development
spec:
  limits:
  - max:
      cpu: "2"
      memory: "2Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    type: Container
```

```bash
# View resource quotas
kubectl get resourcequota -n development
kubectl describe resourcequota compute-quota -n development

# View limit ranges
kubectl get limitrange -n development
kubectl describe limitrange resource-limits -n development

# View resource usage
kubectl top nodes
kubectl top pods
kubectl top pods --containers
kubectl top pods -n development
```

## Health Checks

### Liveness Probes

Checks if container is alive. Restarts container if probe fails.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: webapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
    
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 15
      periodSeconds: 10
      timeoutSeconds: 3
      successThreshold: 1
      failureThreshold: 3
```

### Readiness Probes

Checks if container is ready to serve traffic. Removes from service if probe fails.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - name: webapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
    
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 2
      successThreshold: 1
      failureThreshold: 3
```

### Startup Probes

Checks if application has started. Delays liveness/readiness probes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
spec:
  containers:
  - name: webapp
    image: slow-starting-app:1.0
    
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 10
      timeoutSeconds: 3
      successThreshold: 1
      failureThreshold: 30  # 30 * 10 = 300s max startup time
    
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      periodSeconds: 10
    
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
```

### Probe Types

#### HTTP GET Probe
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    scheme: HTTP  # or HTTPS
```

#### TCP Socket Probe
```yaml
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```

#### Exec Command Probe
```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Complete Example with All Probes

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
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        
        # Startup probe - checks if app started
        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        
        # Liveness probe - checks if app is alive
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 3
        
        # Readiness probe - checks if app is ready for traffic
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 2
          failureThreshold: 3
```

## Do's and Don'ts

### ✅ DO:

1. **Use ConfigMaps for Configuration**
```yaml
# ✅ Good - externalize configuration
envFrom:
- configMapRef:
    name: app-config
```

2. **Use Secrets for Sensitive Data**
```yaml
# ✅ Good
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

3. **Set Resource Requests and Limits**
```yaml
# ✅ Good
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

4. **Implement Health Checks**
```yaml
# ✅ Good
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

5. **Use PersistentVolumeClaims for Data**
```yaml
# ✅ Good
volumes:
- name: data
  persistentVolumeClaim:
    claimName: app-pvc
```

6. **Use Ingress for HTTP Routing**
```yaml
# ✅ Good - single entry point
kind: Ingress
spec:
  rules:
  - host: myapp.example.com
```

7. **Use Rolling Updates**
```yaml
# ✅ Good
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

8. **Monitor Rollout Status**
```bash
kubectl rollout status deployment/webapp
kubectl rollout history deployment/webapp
```

### ❌ DON'T:

1. **Don't Hardcode Configuration**
```yaml
# ❌ Bad
env:
- name: DATABASE_HOST
  value: "192.168.1.100"
```

2. **Don't Store Secrets in Plain Text**
```yaml
# ❌ Bad
env:
- name: PASSWORD
  value: "supersecret123"
```

3. **Don't Skip Resource Limits**
```yaml
# ❌ Bad - can consume all resources
containers:
- name: app
  image: myapp:1.0
  # No resources specified
```

4. **Don't Ignore Health Checks**
```yaml
# ❌ Bad - no health checks
containers:
- name: app
  image: myapp:1.0
  ports:
  - containerPort: 8080
  # No probes
```

5. **Don't Use hostPath in Production**
```yaml
# ❌ Bad - ties Pod to specific node
volumes:
- name: data
  hostPath:
    path: /data
```

6. **Don't Use Recreate Strategy for Critical Apps**
```yaml
# ❌ Bad - causes downtime
strategy:
  type: Recreate
```

7. **Don't Expose Services Without Ingress**
```yaml
# ❌ Bad - each service gets external IP
type: LoadBalancer
```

8. **Don't Forget to Test Rollouts**
```bash
# Always monitor rollouts
kubectl rollout status deployment/webapp
```

## Troubleshooting

### ConfigMap Issues

```bash
# Verify ConfigMap exists
kubectl get configmap app-config

# Check ConfigMap content
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# Verify Pod is using ConfigMap
kubectl describe pod <pod-name> | grep -A5 "Environment\|Mounts"

# Check if ConfigMap is mounted
kubectl exec <pod-name> -- ls -la /config
kubectl exec <pod-name> -- cat /config/key
```

### Secret Issues

```bash
# Verify Secret exists
kubectl get secret db-secret

# Check Secret (base64 encoded)
kubectl get secret db-secret -o yaml

# Decode Secret
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 --decode

# Verify Pod is using Secret
kubectl describe pod <pod-name>
kubectl exec <pod-name> -- env | grep PASSWORD
```

### PVC Issues

```bash
# Check PVC status
kubectl get pvc
kubectl describe pvc app-pvc

# Check if PV is bound
kubectl get pv

# Check StorageClass
kubectl get storageclass

# Pod pending due to PVC
kubectl describe pod <pod-name> | grep -A5 "Events"
```

### Ingress Issues

```bash
# Verify Ingress exists
kubectl get ingress

# Check Ingress details
kubectl describe ingress webapp-ingress

# Verify Ingress controller is running
kubectl get pods -n ingress-nginx

# Check service exists and has endpoints
kubectl get service webapp-service
kubectl get endpoints webapp-service

# Test DNS resolution
nslookup myapp.example.com
```

### Probe Failures

```bash
# Check probe configuration
kubectl describe pod <pod-name> | grep -A10 "Liveness\|Readiness"

# Check probe endpoint manually
kubectl port-forward <pod-name> 8080:8080
curl http://localhost:8080/healthz

# Check events
kubectl get events --field-selector involvedObject.name=<pod-name>

# View logs
kubectl logs <pod-name>
```

## Quick Command Reference

```bash
# ConfigMaps
kubectl create configmap <name> --from-literal=key=value
kubectl get configmap
kubectl describe configmap <name>

# Secrets
kubectl create secret generic <name> --from-literal=key=value
kubectl get secret
kubectl describe secret <name>

# PVC
kubectl get pvc
kubectl describe pvc <name>
kubectl get pv

# Ingress
kubectl get ingress
kubectl describe ingress <name>

# Resource Management
kubectl top nodes
kubectl top pods
kubectl describe resourcequota
kubectl describe limitrange

# Rollouts
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout restart deployment/<name>
```
