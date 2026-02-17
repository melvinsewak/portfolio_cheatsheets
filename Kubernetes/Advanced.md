# Kubernetes Advanced Cheatsheet

## StatefulSets

### What are StatefulSets?

StatefulSets manage stateful applications requiring:
- Stable, unique network identifiers
- Stable, persistent storage
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates

Use cases: Databases, distributed systems, applications requiring stable identity.

### StatefulSet vs Deployment

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| Pod Identity | Random | Stable, ordered (pod-0, pod-1) |
| Storage | Shared or ephemeral | Persistent per Pod |
| Scaling | Parallel | Sequential |
| Network | Random Pod IP | Predictable hostname |
| Use Case | Stateless apps | Stateful apps |

### Basic StatefulSet

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None  # Headless service
  selector:
    app: nginx
  ports:
  - port: 80
    name: web
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx-headless"
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
        image: nginx:alpine
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

### Pod Identity

```bash
# Pods have predictable names
# web-0, web-1, web-2

# Stable network identity
# web-0.nginx-headless.default.svc.cluster.local
# web-1.nginx-headless.default.svc.cluster.local
# web-2.nginx-headless.default.svc.cluster.local

# Test DNS resolution
kubectl run -it dns-test --image=busybox --rm -- nslookup web-0.nginx-headless
```

### StatefulSet with Persistent Storage

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "256Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"
            cpu: "1000m"
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: standard
      resources:
        requests:
          storage: 10Gi
```

### Managing StatefulSets

```bash
# Create StatefulSet
kubectl apply -f statefulset.yaml

# List StatefulSets
kubectl get statefulsets
kubectl get sts

# Get StatefulSet details
kubectl describe statefulset web

# Scale StatefulSet (scales down from highest ordinal)
kubectl scale statefulset web --replicas=5

# Delete StatefulSet (keeps PVCs)
kubectl delete statefulset web

# Delete StatefulSet and PVCs
kubectl delete statefulset web
kubectl delete pvc www-web-0 www-web-1 www-web-2

# Update StatefulSet (rolling update)
kubectl set image statefulset/web nginx=nginx:1.25

# Check rollout status
kubectl rollout status statefulset/web

# Force deletion of stuck Pod
kubectl delete pod web-0 --force --grace-period=0
```

### Update Strategies

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2  # Update pods with ordinal >= 2
  # or
  updateStrategy:
    type: OnDelete  # Manual update by deleting pods
```

## DaemonSets

### What are DaemonSets?

DaemonSets ensure a copy of a Pod runs on all (or selected) nodes. Common uses:
- Node monitoring (Prometheus node exporter)
- Log collection (Fluentd, Logstash)
- Storage daemons (Ceph, Gluster)
- Network proxies (kube-proxy)

### Basic DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:latest
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### DaemonSet with Node Selector

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disktype: ssd  # Only runs on nodes with this label
      containers:
      - name: monitor
        image: monitoring-agent:latest
```

### DaemonSet with Tolerations

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-monitor
spec:
  selector:
    matchLabels:
      app: node-monitor
  template:
    metadata:
      labels:
        app: node-monitor
    spec:
      tolerations:
      # Allows running on master nodes
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      # Allows running on nodes with disk pressure
      - key: node.kubernetes.io/disk-pressure
        operator: Exists
        effect: NoSchedule
      containers:
      - name: monitor
        image: monitoring-agent:latest
        securityContext:
          privileged: true  # Often needed for node-level operations
```

### Managing DaemonSets

```bash
# Create DaemonSet
kubectl apply -f daemonset.yaml

# List DaemonSets
kubectl get daemonsets
kubectl get ds

# Get DaemonSet details
kubectl describe daemonset fluentd -n kube-system

# Update DaemonSet
kubectl set image daemonset/fluentd fluentd=fluent/fluentd:v1.14 -n kube-system

# Check rollout status
kubectl rollout status daemonset/fluentd -n kube-system

# Delete DaemonSet
kubectl delete daemonset fluentd -n kube-system

# View DaemonSet Pods
kubectl get pods -l app=fluentd -n kube-system -o wide
```

## Jobs

### What are Jobs?

Jobs create one or more Pods and ensure they successfully terminate. Use cases:
- Batch processing
- One-time tasks
- Data migration
- Report generation

### Basic Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculation
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:slim
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4  # Number of retries before marking as failed
```

### Parallel Jobs

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  parallelism: 3        # Run 3 pods in parallel
  completions: 9        # Total 9 successful completions needed
  template:
    spec:
      containers:
      - name: worker
        image: busybox
        command: ["sh", "-c", "echo Processing item $ITEM && sleep 10"]
      restartPolicy: Never
  backoffLimit: 5
```

### Job with Persistent Volume

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processor
spec:
  template:
    spec:
      containers:
      - name: processor
        image: data-processor:latest
        command: ["python", "process.py"]
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: data-pvc
      restartPolicy: Never
  backoffLimit: 3
```

### Job with TTL (Auto-cleanup)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: cleanup-job
spec:
  ttlSecondsAfterFinished: 3600  # Delete job 1 hour after completion
  template:
    spec:
      containers:
      - name: cleanup
        image: cleanup-tool:latest
        command: ["./cleanup.sh"]
      restartPolicy: Never
```

### Managing Jobs

```bash
# Create Job
kubectl apply -f job.yaml

# List Jobs
kubectl get jobs

# Get Job details
kubectl describe job pi-calculation

# View Job logs
kubectl logs job/pi-calculation

# View Job Pods
kubectl get pods --selector=job-name=pi-calculation

# Wait for Job completion
kubectl wait --for=condition=complete job/pi-calculation --timeout=300s

# Delete Job and its Pods
kubectl delete job pi-calculation

# Delete completed Jobs
kubectl delete jobs --field-selector status.successful=1
```

## CronJobs

### What are CronJobs?

CronJobs run Jobs on a time-based schedule. Use cases:
- Scheduled backups
- Report generation
- Database maintenance
- Data synchronization

### Basic CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"  # Run at 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
            command: ["./backup.sh"]
            env:
            - name: BACKUP_LOCATION
              value: "/backups"
            volumeMounts:
            - name: backup-storage
              mountPath: /backups
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3  # Keep last 3 successful jobs
  failedJobsHistoryLimit: 1       # Keep last 1 failed job
```

### CronJob Schedule Examples

```yaml
# Every minute
schedule: "* * * * *"

# Every hour at minute 30
schedule: "30 * * * *"

# Every day at 2:30 AM
schedule: "30 2 * * *"

# Every Monday at 9 AM
schedule: "0 9 * * 1"

# First day of every month at midnight
schedule: "0 0 1 * *"

# Every 15 minutes
schedule: "*/15 * * * *"

# Weekdays at 6 PM
schedule: "0 18 * * 1-5"
```

### CronJob with Concurrency Control

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: data-sync
spec:
  schedule: "*/10 * * * *"  # Every 10 minutes
  concurrencyPolicy: Forbid  # Don't allow concurrent runs
  # Options: Allow, Forbid, Replace
  startingDeadlineSeconds: 300  # Must start within 5 minutes
  suspend: false  # Set to true to suspend CronJob
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: sync
            image: sync-tool:latest
            command: ["./sync.sh"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 5
  failedJobsHistoryLimit: 3
```

### Managing CronJobs

```bash
# Create CronJob
kubectl apply -f cronjob.yaml

# List CronJobs
kubectl get cronjobs
kubectl get cj

# Get CronJob details
kubectl describe cronjob backup-job

# View Jobs created by CronJob
kubectl get jobs --selector=cronjob=backup-job

# Suspend CronJob (pause)
kubectl patch cronjob backup-job -p '{"spec":{"suspend":true}}'

# Resume CronJob
kubectl patch cronjob backup-job -p '{"spec":{"suspend":false}}'

# Manually trigger CronJob
kubectl create job --from=cronjob/backup-job manual-backup-001

# Delete CronJob
kubectl delete cronjob backup-job

# Delete CronJob and all its Jobs
kubectl delete cronjob backup-job --cascade=foreground
```

## RBAC (Role-Based Access Control)

### RBAC Components

- **ServiceAccount**: Identity for processes running in Pods
- **Role**: Namespace-scoped permissions
- **ClusterRole**: Cluster-wide permissions
- **RoleBinding**: Grants Role permissions to users/groups/ServiceAccounts in a namespace
- **ClusterRoleBinding**: Grants ClusterRole permissions cluster-wide

### ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: default
```

```bash
# Create ServiceAccount
kubectl create serviceaccount app-service-account

# List ServiceAccounts
kubectl get serviceaccounts
kubectl get sa

# Get ServiceAccount token
kubectl create token app-service-account

# View ServiceAccount
kubectl describe serviceaccount app-service-account
```

### Role (Namespace-scoped)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]  # "" indicates core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
```

### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRole (Cluster-wide)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
```

### ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: ServiceAccount
  name: monitoring-sa
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

### Common RBAC Verbs

```yaml
verbs:
- get        # Read single resource
- list       # List resources
- watch      # Watch for changes
- create     # Create resources
- update     # Update existing resources
- patch      # Patch resources
- delete     # Delete resources
- deletecollection  # Delete multiple resources
```

### Pod Using ServiceAccount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  serviceAccountName: app-service-account
  containers:
  - name: app
    image: myapp:1.0
```

### Complete RBAC Example

```yaml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: deployment-manager
  namespace: production
---
# Role - manage deployments
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager-role
  namespace: production
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployment-manager-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: deployment-manager
  namespace: production
roleRef:
  kind: Role
  name: deployment-manager-role
  apiGroup: rbac.authorization.k8s.io
---
# Deployment using ServiceAccount
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: production
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
      serviceAccountName: deployment-manager
      containers:
      - name: webapp
        image: myapp:1.0
```

### Testing RBAC

```bash
# Check if you can perform action
kubectl auth can-i create deployments --namespace=production

# Check as specific user
kubectl auth can-i list pods --as=system:serviceaccount:default:app-service-account

# Check all permissions for ServiceAccount
kubectl auth can-i --list --as=system:serviceaccount:default:app-service-account

# View Role
kubectl get role pod-reader -o yaml

# View RoleBinding
kubectl get rolebinding read-pods-binding -o yaml

# List all Roles in namespace
kubectl get roles -n production

# List all ClusterRoles
kubectl get clusterroles
```

## Helm Basics

### What is Helm?

Helm is a package manager for Kubernetes that:
- Packages Kubernetes resources as Charts
- Manages application releases
- Supports templating and values
- Enables version control and rollbacks

### Installing Helm

```bash
# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# macOS
brew install helm

# Windows
choco install kubernetes-helm

# Verify installation
helm version
```

### Helm Repositories

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable

# Update repositories
helm repo update

# List repositories
helm repo list

# Search for charts
helm search repo nginx
helm search repo bitnami/

# Remove repository
helm repo remove bitnami
```

### Installing Charts

```bash
# Install chart
helm install my-nginx bitnami/nginx

# Install with custom name
helm install webapp bitnami/nginx

# Install with custom values
helm install webapp bitnami/nginx --set replicaCount=3

# Install with values file
helm install webapp bitnami/nginx -f values.yaml

# Install in specific namespace
helm install webapp bitnami/nginx -n production --create-namespace

# Dry run (see what would be installed)
helm install webapp bitnami/nginx --dry-run --debug

# Generate manifests without installing
helm template webapp bitnami/nginx > manifests.yaml
```

### Managing Releases

```bash
# List releases
helm list
helm list -A  # All namespaces
helm list -n production

# Get release status
helm status webapp

# View release history
helm history webapp

# Upgrade release
helm upgrade webapp bitnami/nginx --set replicaCount=5

# Upgrade with new values file
helm upgrade webapp bitnami/nginx -f new-values.yaml

# Rollback to previous revision
helm rollback webapp

# Rollback to specific revision
helm rollback webapp 2

# Uninstall release
helm uninstall webapp

# Uninstall but keep history
helm uninstall webapp --keep-history
```

### Custom Values File

**values.yaml:**
```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent

service:
  type: LoadBalancer
  port: 80

resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"

ingress:
  enabled: true
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

### Creating Your Own Chart

```bash
# Create new chart
helm create mychart

# Chart structure
# mychart/
#   Chart.yaml          # Chart metadata
#   values.yaml         # Default values
#   charts/             # Dependency charts
#   templates/          # Kubernetes manifests
#     deployment.yaml
#     service.yaml
#     ingress.yaml
#     _helpers.tpl      # Template helpers

# Validate chart
helm lint mychart

# Package chart
helm package mychart

# Install local chart
helm install myapp ./mychart
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: mychart
description: A Helm chart for my application
type: application
version: 1.0.0
appVersion: "1.0"
```

**templates/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.port }}
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

## Network Policies

### What are Network Policies?

Network Policies control traffic between Pods and network endpoints. They act as a firewall for Kubernetes clusters.

### Default Deny All Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}  # Applies to all Pods in namespace
  policyTypes:
  - Ingress
```

### Allow Specific Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Allow from Specific Namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-namespace
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production
    ports:
    - protocol: TCP
      port: 8080
```

### Egress Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
spec:
  podSelector:
    matchLabels:
      app: webapp
  policyTypes:
  - Egress
  egress:
  # Allow DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
  # Allow external HTTPS
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
    ports:
    - protocol: TCP
      port: 443
```

### Complete Network Policy Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: webapp-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: webapp
  policyTypes:
  - Ingress
  - Egress
  
  ingress:
  # Allow from frontend
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  
  # Allow from monitoring namespace
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 9090
  
  egress:
  # Allow DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
  
  # Allow to database
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
  
  # Allow to external API
  - to:
    - ipBlock:
        cidr: 203.0.113.0/24
    ports:
    - protocol: TCP
      port: 443
```

### Managing Network Policies

```bash
# Create NetworkPolicy
kubectl apply -f networkpolicy.yaml

# List NetworkPolicies
kubectl get networkpolicies
kubectl get netpol

# Get NetworkPolicy details
kubectl describe networkpolicy webapp-network-policy

# Delete NetworkPolicy
kubectl delete networkpolicy webapp-network-policy

# Test connectivity (from Pod)
kubectl exec -it frontend-pod -- curl http://backend-service:8080
```

## Monitoring and Logging

### Metrics Server

```bash
# Install Metrics Server (required for kubectl top)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# For minikube
minikube addons enable metrics-server

# Verify installation
kubectl get deployment metrics-server -n kube-system

# View node metrics
kubectl top nodes

# View pod metrics
kubectl top pods
kubectl top pods -n kube-system
kubectl top pods --containers
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory
```

### Resource Monitoring

```bash
# Node resource usage
kubectl top nodes

# Pod resource usage
kubectl top pods --all-namespaces

# Pod resource usage with containers
kubectl top pods --containers -n production

# Sort by CPU
kubectl top pods --sort-by=cpu -A

# Sort by memory
kubectl top pods --sort-by=memory -A

# Watch resource usage
watch kubectl top pods -n production
```

### Logging

```bash
# View Pod logs
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>

# Logs from specific container
kubectl logs <pod-name> -c <container-name>

# Previous container logs (after restart)
kubectl logs <pod-name> --previous

# Logs with timestamps
kubectl logs <pod-name> --timestamps

# Last N lines
kubectl logs <pod-name> --tail=100

# Logs from all containers in Pod
kubectl logs <pod-name> --all-containers=true

# Logs from Deployment
kubectl logs deployment/<deployment-name>

# Logs from multiple Pods (label selector)
kubectl logs -l app=webapp --all-containers=true

# Stream logs from all Pods in Deployment
kubectl logs -f deployment/webapp --all-containers=true
```

### Events

```bash
# Get cluster events
kubectl get events

# Watch events
kubectl get events --watch

# Events sorted by timestamp
kubectl get events --sort-by=.metadata.creationTimestamp

# Events for specific resource
kubectl get events --field-selector involvedObject.name=<pod-name>

# Events in specific namespace
kubectl get events -n production

# Events for specific type
kubectl get events --field-selector type=Warning
```

### Prometheus and Grafana (Monitoring Stack)

```bash
# Install Prometheus using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Access Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Get Grafana admin password
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

## Troubleshooting

### Common Issues

#### Pod Stuck in Pending
```bash
# Check events
kubectl describe pod <pod-name>

# Common causes:
# 1. Insufficient resources
kubectl top nodes
kubectl describe node <node-name>

# 2. PVC not bound
kubectl get pvc

# 3. Node selector mismatch
kubectl get nodes --show-labels
```

#### Pod CrashLoopBackOff
```bash
# Check logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Check container exit code
kubectl describe pod <pod-name> | grep -A5 "State"

# Common causes:
# - Application error
# - Missing configuration
# - Failed health checks
# - Permission issues
```

#### ImagePullBackOff
```bash
# Check image details
kubectl describe pod <pod-name> | grep -A5 "Events"

# Common causes:
# - Wrong image name/tag
# - No imagePullSecret for private registry
# - Registry unavailable

# Check imagePullSecret
kubectl get secret <secret-name> -o yaml
```

#### Service Not Accessible
```bash
# Check service
kubectl get service <service-name>

# Check endpoints
kubectl get endpoints <service-name>

# If no endpoints, check:
# 1. Pod labels match service selector
kubectl get pods --show-labels
kubectl describe service <service-name>

# 2. Pods are running and ready
kubectl get pods -l app=webapp

# Test connectivity from another Pod
kubectl run test --image=busybox -it --rm -- wget -O- http://service-name
```

#### Node NotReady
```bash
# Check node status
kubectl get nodes
kubectl describe node <node-name>

# Check node conditions
kubectl get node <node-name> -o yaml | grep -A10 conditions

# SSH to node and check
# - Disk space
# - Memory
# - kubelet status: systemctl status kubelet
# - Container runtime: systemctl status containerd
```

### Debug Commands

```bash
# Run debug Pod
kubectl run debug --image=busybox:latest -it --rm -- sh

# Run with curl
kubectl run curl-test --image=curlimages/curl -it --rm -- sh

# Run with network tools
kubectl run nettools --image=nicolaka/netshoot -it --rm -- bash

# Debug existing Pod
kubectl debug <pod-name> -it --image=busybox

# Copy Pod for debugging
kubectl debug <pod-name> -it --copy-to=debug-pod --container=app

# Execute in running container
kubectl exec -it <pod-name> -- bash

# Check container filesystem
kubectl exec <pod-name> -- df -h
kubectl exec <pod-name> -- ls -la /app
```

## Do's and Don'ts

### ✅ DO:

1. **Use StatefulSets for Stateful Apps**
```yaml
# ✅ Good for databases
kind: StatefulSet
spec:
  volumeClaimTemplates:
  - metadata:
      name: data
```

2. **Implement RBAC**
```yaml
# ✅ Good - least privilege
kind: Role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

3. **Use Network Policies**
```yaml
# ✅ Good - restrict network access
kind: NetworkPolicy
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

4. **Use DaemonSets for Node-Level Services**
```yaml
# ✅ Good for monitoring agents
kind: DaemonSet
spec:
  template:
    spec:
      containers:
      - name: node-exporter
```

5. **Use Helm for Complex Applications**
```bash
# ✅ Good - version control and templating
helm install myapp ./chart -f values.yaml
```

6. **Monitor Resources**
```bash
kubectl top nodes
kubectl top pods -A
```

7. **Use Jobs for One-Time Tasks**
```yaml
# ✅ Good
kind: Job
spec:
  template:
    spec:
      restartPolicy: Never
```

8. **Use CronJobs for Scheduled Tasks**
```yaml
# ✅ Good
kind: CronJob
spec:
  schedule: "0 2 * * *"
```

### ❌ DON'T:

1. **Don't Use Deployments for Stateful Apps**
```yaml
# ❌ Bad for databases
kind: Deployment  # Use StatefulSet instead
```

2. **Don't Grant Excessive Permissions**
```yaml
# ❌ Bad
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

3. **Don't Skip Network Policies**
```yaml
# ❌ Bad - no network restrictions
# Missing NetworkPolicy
```

4. **Don't Use Jobs for Long-Running Services**
```yaml
# ❌ Bad - use Deployment instead
kind: Job
spec:
  template:
    spec:
      containers:
      - name: webserver  # Should be Deployment
```

5. **Don't Hardcode Values in Charts**
```yaml
# ❌ Bad
image: myapp:1.0  # Should use {{ .Values.image }}
```

6. **Don't Ignore Resource Limits in StatefulSets**
```yaml
# ❌ Bad
spec:
  template:
    spec:
      containers:
      - name: db
        # No resource limits
```

7. **Don't Delete StatefulSet PVCs Accidentally**
```bash
# ❌ Bad - loses data
kubectl delete statefulset web --cascade=foreground
kubectl delete pvc --all  # Don't do this!
```

8. **Don't Run Privileged Containers Without Need**
```yaml
# ❌ Bad unless absolutely necessary
securityContext:
  privileged: true
```

## Security Summary

When implementing advanced Kubernetes features, always consider:

1. **RBAC**: Use least privilege principle
2. **Network Policies**: Restrict traffic between Pods
3. **SecurityContext**: Run containers as non-root
4. **Pod Security Standards**: Enforce security policies
5. **Secrets**: Never commit secrets to Git
6. **Image Scanning**: Scan images for vulnerabilities
7. **Resource Limits**: Prevent resource exhaustion
8. **Audit Logging**: Enable audit logs for compliance

## Quick Command Reference

```bash
# StatefulSets
kubectl get statefulsets
kubectl scale statefulset <name> --replicas=5
kubectl rollout status statefulset/<name>

# DaemonSets
kubectl get daemonsets -A
kubectl rollout status daemonset/<name>

# Jobs
kubectl get jobs
kubectl logs job/<name>
kubectl delete jobs --field-selector status.successful=1

# CronJobs
kubectl get cronjobs
kubectl create job --from=cronjob/<name> manual-job
kubectl patch cronjob <name> -p '{"spec":{"suspend":true}}'

# RBAC
kubectl get serviceaccounts
kubectl get roles
kubectl auth can-i <verb> <resource>

# Helm
helm list
helm install <name> <chart>
helm upgrade <name> <chart>
helm rollback <name>

# Network Policies
kubectl get networkpolicies
kubectl describe netpol <name>

# Monitoring
kubectl top nodes
kubectl top pods -A
kubectl logs -f deployment/<name>
```
