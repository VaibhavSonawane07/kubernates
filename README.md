# Kubernetes.

## Table of Contents
1. [What is Kubernetes?](#what-is-kubernetes)
2. [Core Components](#core-components)
3. [Pod](#pod)
4. [Service](#service)
5. [Deployment](#deployment)
6. [ConfigMap & Secrets](#configmap--secrets)
7. [Namespaces](#namespaces)
8. [Ingress](#ingress)
9. [Volumes & Persistent Volumes](#volumes--persistent-volumes)
10. [RBAC (Role-Based Access Control)](#rbac-role-based-access-control)
11. [Monitoring & Logging](#monitoring--logging)
12. [Troubleshooting](#troubleshooting)
13. [Common Interview Questions](#common-interview-questions)

---

## What is Kubernetes?

**Kubernetes (K8s)** is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

### Key Features:
- **Container Orchestration**: Manages containers across multiple hosts
- **Service Discovery**: Automatic discovery and load balancing
- **Storage Orchestration**: Mounts storage systems automatically
- **Automated Rollouts/Rollbacks**: Manages application updates
- **Self-healing**: Restarts failed containers, replaces nodes
- **Secret & Configuration Management**: Manages sensitive data

---

## Core Components

### Master Node Components:
- **kube-apiserver**: Entry point for all REST commands
- **etcd**: Distributed key-value store for cluster data
- **kube-scheduler**: Assigns pods to nodes
- **kube-controller-manager**: Manages controllers (replication, endpoints, etc.)

### Worker Node Components:
- **kubelet**: Agent that communicates with master
- **kube-proxy**: Network proxy for services
- **Container Runtime**: Docker, containerd, or CRI-O

---

## Pod
![alt text](/assets/Pod.png)

**Pod** is the smallest deployable unit in Kubernetes. It represents a single instance of a running process.

### Key Characteristics:
- Contains one or more containers
- Shares storage and network
- Has a unique IP address
- Containers in a pod share localhost

### Pod Lifecycle:
1. **Pending**: Pod accepted but not scheduled
2. **Running**: Pod bound to node, containers running
3. **Succeeded**: All containers terminated successfully
4. **Failed**: All containers terminated, at least one failed
5. **Unknown**: Pod state cannot be obtained

### Common Pod Commands:
```bash
# Create a pod
kubectl run nginx --image=nginx

# Get pods
kubectl get pods

# Describe pod
kubectl describe pod <pod-name>

# Delete pod
kubectl delete pod <pod-name>

# Get pod logs
kubectl logs <pod-name>
```

### Sample Pod YAML:
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
    image: nginx:1.20
    ports:
    - containerPort: 80
```

---

## Service
![alt text](/assets/Service.png)

**Service** provides stable network endpoint to access pods. It enables communication between different parts of an application.

### Service Types:
1. **ClusterIP** (Default): Internal cluster communication only
2. **NodePort**: Exposes service on each node's IP at a static port
3. **LoadBalancer**: Exposes service externally using cloud provider's load balancer
4. **ExternalName**: Maps service to DNS name

### Service Discovery:
- **Environment Variables**: Kubernetes injects service info as env vars
- **DNS**: Cluster DNS automatically creates DNS records for services

### Sample Service YAML:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

---

## Deployment

**Deployment** manages a set of identical pods, ensuring desired state and enabling rolling updates.

### Key Features:
- **Desired State Management**: Maintains specified number of replicas
- **Rolling Updates**: Zero-downtime updates
- **Rollback Capability**: Can revert to previous versions
- **Scaling**: Easy horizontal scaling

### Sample Deployment YAML:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx:1.20
        ports:
        - containerPort: 80
```

### Deployment Commands:
```bash
# Create deployment
kubectl create deployment nginx --image=nginx

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Update deployment
kubectl set image deployment/nginx nginx=nginx:1.21

# Rollback deployment
kubectl rollout undo deployment/nginx

# Check rollout status
kubectl rollout status deployment/nginx
```

---

## ConfigMap & Secrets

### ConfigMap
Stores non-confidential configuration data in key-value pairs.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "mysql://localhost:3306/mydb"
  debug_mode: "true"
```

### Secrets
Stores sensitive data like passwords, tokens, and keys.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: MWYyZDFlMmU2N2Rm  # base64 encoded
```

### Usage in Pods:
```yaml
spec:
  containers:
  - name: app
    image: myapp
    env:
    - name: DB_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
```

---

## Namespaces

**Namespaces** provide a way to divide cluster resources between multiple users or teams.

### Default Namespaces:
- **default**: Default namespace for objects with no namespace
- **kube-system**: System components
- **kube-public**: Public resources
- **kube-node-lease**: Node heartbeat data

### Namespace Commands:
```bash
# Create namespace
kubectl create namespace development

# List namespaces
kubectl get namespaces

# Set default namespace
kubectl config set-context --current --namespace=development

# Delete namespace
kubectl delete namespace development
```

---

## Ingress
**Ingress** manages external access to services, typically HTTP/HTTPS routing.

### Features:
- **Host-based Routing**: Route traffic based on hostname
- **Path-based Routing**: Route traffic based on URL path
- **TLS Termination**: Handle SSL/TLS certificates
- **Load Balancing**: Distribute traffic across multiple services

### Sample Ingress YAML:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
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
              number: 80
```

---

## Volumes & Persistent Volumes

### Volume Types:
- **emptyDir**: Temporary directory that shares pod's lifetime
- **hostPath**: Mounts directory from host node
- **configMap/secret**: Mount configuration data
- **persistentVolumeClaim**: Request for persistent storage

### Persistent Volume (PV):
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /data
```

### Persistent Volume Claim (PVC):
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

---

## RBAC (Role-Based Access Control)

Controls access to Kubernetes API based on roles assigned to users.

### Components:
- **Role/ClusterRole**: Defines permissions
- **RoleBinding/ClusterRoleBinding**: Binds roles to users/groups
- **ServiceAccount**: Identity for processes running in pods

### Sample Role:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### Sample RoleBinding:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## Monitoring & Logging

### Health Checks:
- **Liveness Probe**: Determines if container is running
- **Readiness Probe**: Determines if container is ready to serve traffic
- **Startup Probe**: Determines if container application has started

### Sample Probes:
```yaml
spec:
  containers:
  - name: app
    image: myapp
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

---

## Troubleshooting

### Common Commands:
```bash
# Check cluster info
kubectl cluster-info

# Check node status
kubectl get nodes

# Describe resources
kubectl describe <resource-type> <resource-name>

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check logs
kubectl logs <pod-name> -c <container-name>

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/bash

# Port forward
kubectl port-forward <pod-name> 8080:80

# Check resource usage
kubectl top nodes
kubectl top pods
```

### Common Issues:
1. **ImagePullBackOff**: Cannot pull container image
2. **CrashLoopBackOff**: Container keeps crashing
3. **Pending Pods**: Insufficient resources or scheduling issues
4. **Service Not Accessible**: Check selectors and endpoints

---

## Common Interview Questions

### 1. **What is the difference between a Pod and a Container?**
- Container is a lightweight, standalone package that includes everything needed to run an application
- Pod is a Kubernetes abstraction that wraps one or more containers with shared storage and network

### 2. **Explain the difference between Deployment and StatefulSet?**
- **Deployment**: For stateless applications, pods are interchangeable
- **StatefulSet**: For stateful applications, maintains persistent identity and ordered deployment

### 3. **What is the difference between ConfigMap and Secret?**
- **ConfigMap**: Stores non-confidential configuration data
- **Secret**: Stores sensitive data with base64 encoding and additional security features

### 4. **How does Service Discovery work in Kubernetes?**
- Environment variables injected into pods
- DNS-based service discovery through cluster DNS
- Services get stable DNS names and IP addresses

### 5. **What is the difference between ClusterIP, NodePort, and LoadBalancer?**
- **ClusterIP**: Internal cluster access only
- **NodePort**: Exposes service on each node's IP at a static port
- **LoadBalancer**: Uses cloud provider's load balancer for external access

### 6. **How do you perform rolling updates?**
```bash
kubectl set image deployment/myapp container=image:new-version
kubectl rollout status deployment/myapp
kubectl rollout undo deployment/myapp  # if needed
```

### 7. **What are Kubernetes Operators?**
Custom controllers that extend Kubernetes API to manage complex applications using domain-specific knowledge.

### 8. **Explain Pod lifecycle and restart policies?**
- **Always**: Always restart container on failure (default)
- **OnFailure**: Restart only if container exits with error
- **Never**: Never restart container

### 9. **What is a DaemonSet?**
Ensures that all (or some) nodes run a copy of a pod. Used for system daemons like log collectors, monitoring agents.

### 10. **How do you handle secrets securely?**
- Use Kubernetes Secrets instead of hardcoding
- Enable encryption at rest
- Use RBAC to limit access
- Consider external secret management tools (Vault, AWS Secrets Manager)

---
