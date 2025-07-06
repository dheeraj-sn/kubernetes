# 🚀 Kubernetes Learning Journal - Phase 1 Complete!

## 📅 Session Date: July 6, 2025
**Progress**: Phase 1 Complete ✅ (Steps 1-3)

---

## 🎯 What We Accomplished Today

### Phase 1, Step 1: Setup Local Kubernetes Environment ✅

#### 1.1 Environment Verification
- **Checked kubectl installation**: Found at `/usr/local/bin/kubectl`
- **Verified kubectl version**: v1.32.2 (recent version)
- **Confirmed Docker availability**: Docker v28.1.1 installed

#### 1.2 Kubernetes Cluster Setup
- **Started Docker Desktop**: Enabled Kubernetes in Docker Desktop settings
- **Verified cluster status**: 
  ```bash
  kubectl cluster-info
  # Output: Kubernetes control plane running at https://127.0.0.1:6443
  ```
- **Checked cluster nodes**:
  ```bash
  kubectl get nodes
  # Output: docker-desktop (Ready, control-plane, v1.32.2)
  ```

#### 1.3 System Components Verification
- **Verified system pods**:
  ```bash
  kubectl get pods --all-namespaces
  ```
  **Running components**:
  - CoreDNS (2 pods) - DNS service for service discovery
  - etcd - Key-value store for cluster data
  - kube-apiserver - Kubernetes API server
  - kube-controller-manager - Manages controllers
  - kube-scheduler - Schedules pods to nodes
  - kube-proxy - Network proxy
  - storage-provisioner - Handles storage
  - vpnkit-controller - Network controller

---

### Phase 1, Step 2: Core Concepts - Pods ✅

#### 2.1 Understanding Pods
**What is a Pod?**
- Smallest deployable unit in Kubernetes
- Contains one or more containers
- Shares network namespace and storage
- Basic building block of Kubernetes applications

#### 2.2 Creating Your First Pod
**File created**: `first-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world-pod
  labels:
    app: hello-world
    environment: learning
spec:
  containers:
  - name: nginx-container
    image: nginx:alpine
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "50m"
      limits:
        memory: "128Mi"
        cpu: "100m"
```

#### 2.3 YAML Structure Explained
- **apiVersion**: Kubernetes API version (`v1` for core resources)
- **kind**: Resource type (`Pod`)
- **metadata**: Pod information (name, labels, namespace)
- **spec**: Pod specification (containers, resources, volumes)

#### 2.4 Deploying the Pod
```bash
kubectl apply -f first-pod.yaml
# Output: pod/hello-world-pod created
```

#### 2.5 Pod Lifecycle Management
**Checking pod status**:
```bash
kubectl get pods
# Output: hello-world-pod   1/1     Running   0          24s
```

**Detailed pod information**:
```bash
kubectl describe pod hello-world-pod
```

**Key information from describe**:
- **Pod IP**: 10.1.0.6 (internal cluster IP)
- **Node**: docker-desktop/192.168.65.3
- **Container**: nginx-container (Running, Ready)
- **Resources**: CPU 50m-100m, Memory 64Mi-128Mi
- **Port**: 80/TCP

#### 2.6 Pod Events Timeline
1. **Scheduled**: Pod assigned to docker-desktop node
2. **Pulling**: Downloaded nginx:alpine image (4.477s)
3. **Created**: Container created successfully
4. **Started**: Container started and ready

#### 2.7 Testing the Pod
**Port forwarding**:
```bash
kubectl port-forward hello-world-pod 8080:80
# Output: Forwarding from 127.0.0.1:8080 -> 80
```

**Testing connectivity**:
```bash
curl -s http://localhost:8080 | head -10
# Output: nginx welcome page HTML
```

---

### Phase 1, Step 3: Core Concepts - Services ✅

#### 3.1 Understanding Services
**What is a Service?**
- Provides stable networking for pods
- Enables service discovery and load balancing
- Solves the problem of changing pod IP addresses
- Acts as a stable endpoint for applications

#### 3.2 Service Types Overview
- **ClusterIP**: Internal-only access (default)
- **NodePort**: External access via node port
- **LoadBalancer**: External access via cloud load balancer

#### 3.3 Creating Your First Service (NodePort)
**File created**: `first-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
  labels:
    app: hello-world
    environment: learning
spec:
  type: NodePort
  selector:
    app: hello-world
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
  sessionAffinity: None
```

#### 3.4 Service YAML Structure Explained
- **apiVersion**: Kubernetes API version (`v1`)
- **kind**: Resource type (`Service`)
- **type**: Service type (NodePort, ClusterIP, LoadBalancer)
- **selector**: Label selector to find pods
- **ports**: Port mapping configuration

#### 3.5 Deploying the Service
```bash
kubectl apply -f first-service.yaml
# Output: service/hello-world-service created
```

#### 3.6 Service Status and Details
**Checking service status**:
```bash
kubectl get services
# Output: hello-world-service   NodePort    10.108.176.129   <none>        80:30080/TCP
```

**Detailed service information**:
```bash
kubectl describe service hello-world-service
```

**Key information from describe**:
- **Service IP**: 10.108.176.129 (stable internal IP)
- **Endpoints**: 10.1.0.6:80 (pod IP and port)
- **NodePort**: 30080 (external access port)
- **Selector**: app=hello-world (finds matching pods)

#### 3.7 Testing Service Access
**External access via NodePort**:
```bash
curl -s http://localhost:30080 | head -10
# Output: nginx welcome page HTML
```

#### 3.8 Creating ClusterIP Service
**File created**: `clusterip-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-clusterip
  labels:
    app: hello-world
    environment: learning
spec:
  type: ClusterIP
  selector:
    app: hello-world
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
```

**Deploying ClusterIP service**:
```bash
kubectl apply -f clusterip-service.yaml
# Output: service/hello-world-clusterip created
```

#### 3.9 Testing Internal Service Access
**Internal access from within cluster**:
```bash
kubectl run test-pod --image=busybox --rm -it --restart=Never -- sh -c "wget -qO- http://hello-world-clusterip:8080 | head -5"
# Output: nginx welcome page HTML
```

#### 3.10 Multi-Pod Load Balancing
**Creating deployment with multiple pods**:
**File created**: `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    environment: learning
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        environment: learning
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
```

**Deploying multi-pod application**:
```bash
kubectl apply -f nginx-deployment.yaml
# Output: deployment.apps/nginx-deployment created
```

**Verifying multiple pods**:
```bash
kubectl get pods
# Output: 3 nginx-deployment pods + 1 hello-world-pod
```

#### 3.11 Service for Multiple Pods
**File created**: `nginx-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
    environment: learning
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30081
    protocol: TCP
```

**Deploying service for multiple pods**:
```bash
kubectl apply -f nginx-service.yaml
# Output: service/nginx-service created
```

#### 3.12 Load Balancing Verification
**Checking service endpoints**:
```bash
kubectl describe service nginx-service
# Output: Endpoints: 10.1.0.9:80,10.1.0.10:80,10.1.0.8:80
```

**Testing load balancing**:
```bash
curl -s http://localhost:30081 | head -5
# Output: nginx welcome page HTML
```

---

## 🎓 Key Concepts Learned

### 1. Kubernetes Architecture Basics
- **Control Plane**: Manages the cluster (API server, scheduler, controller manager)
- **Nodes**: Worker machines that run your applications
- **Pods**: Smallest deployable units containing containers
- **Services**: Stable networking layer for pod communication

### 2. kubectl Commands Mastered
- `kubectl cluster-info` - Check cluster status
- `kubectl get nodes` - List cluster nodes
- `kubectl get pods` - List pods in current namespace
- `kubectl get pods --all-namespaces` - List all pods
- `kubectl describe pod <name>` - Detailed pod information
- `kubectl apply -f <file>` - Create/update resources from YAML
- `kubectl port-forward <pod> <local-port>:<container-port>` - Port forwarding
- `kubectl get services` - List services
- `kubectl describe service <name>` - Detailed service information
- `kubectl run <name> --image=<image> --rm -it --restart=Never -- <command>` - Run temporary pod

### 3. Pod Concepts
- **Pod Lifecycle**: Creation, scheduling, running, termination
- **Resource Management**: CPU and memory requests/limits
- **Container Images**: How Kubernetes pulls and manages images
- **Pod Networking**: Internal IP assignment and port exposure
- **Labels**: Key-value pairs for organizing and selecting resources

### 4. Service Concepts
- **Service Types**: ClusterIP, NodePort, LoadBalancer
- **Service Discovery**: How services find pods using labels
- **Load Balancing**: Automatic distribution across multiple pods
- **Port Mapping**: Service port, target port, node port
- **DNS Resolution**: Automatic service names within cluster
- **Endpoint Management**: Automatic pod IP mapping

### 5. YAML Configuration
- **Structure**: apiVersion, kind, metadata, spec
- **Best Practices**: Proper indentation, meaningful names, resource limits
- **Validation**: kubectl validates YAML before applying
- **Service Configuration**: Selectors, ports, session affinity

---

## 🔧 Technical Details

### Cluster Information
- **Cluster Type**: Docker Desktop Kubernetes
- **Kubernetes Version**: v1.32.2
- **Node Count**: 1 (docker-desktop)
- **Architecture**: Single-node cluster for development

### Resource Usage
- **Pod Resources**: 64Mi-128Mi RAM, 50m-100m CPU
- **Image Size**: nginx:alpine (~22MB)
- **Startup Time**: ~5 seconds (including image pull)

### Networking
- **Pod IP Range**: 10.1.0.0/16
- **Service IP Range**: 10.96.0.0/12
- **Port Forwarding**: Local 8080 → Pod 80
- **NodePort Range**: 30000-32767

### Services Created
1. **hello-world-service** (NodePort): 10.108.176.129:80 → 30080
2. **hello-world-clusterip** (ClusterIP): 10.110.237.121:8080 (internal only)
3. **nginx-service** (NodePort): 10.100.54.76:80 → 30081 (load balanced)

---

## 📝 Next Steps Preview

### Phase 2: Application Deployment
- **Deployments**: Rolling updates, scaling, rollbacks
- **ConfigMaps and Secrets**: Configuration management
- **Health Checks**: Liveness and readiness probes

### Phase 3: Advanced Concepts
- **Namespaces and Resource Management**
- **Storage**: Persistent Volumes and Claims
- **Networking Deep Dive**: Ingress controllers

---

## 🎯 Learning Outcomes Achieved

✅ **Environment Setup**: Local Kubernetes cluster running  
✅ **Basic Commands**: Essential kubectl operations mastered  
✅ **Pod Creation**: First application deployed successfully  
✅ **YAML Understanding**: Configuration file structure learned  
✅ **Resource Management**: CPU/memory limits implemented  
✅ **Networking Basics**: Port forwarding and connectivity tested  
✅ **Troubleshooting**: Pod status and event monitoring  
✅ **Service Creation**: Multiple service types implemented  
✅ **Load Balancing**: Multi-pod service with automatic distribution  
✅ **Service Discovery**: Label-based pod selection mastered  
✅ **Internal/External Access**: Both ClusterIP and NodePort services tested  

---

## 💡 Tips & Best Practices Learned

1. **Always check cluster status** before deploying
2. **Use meaningful labels** for better resource organization
3. **Set resource limits** to prevent resource exhaustion
4. **Monitor pod events** for troubleshooting
5. **Test connectivity** after deployment
6. **Use port-forwarding** for development and debugging
7. **Choose appropriate service types** for your use case
8. **Verify service endpoints** to ensure proper pod selection
9. **Test load balancing** with multiple requests
10. **Use DNS names** for internal service communication

---

## 🏆 Phase 1 Complete!

**Congratulations!** You've successfully completed the foundation phase of Kubernetes learning. You now have a solid understanding of:

- ✅ **Pods**: The basic building blocks
- ✅ **Services**: Networking and load balancing
- ✅ **Basic kubectl operations**: Essential commands
- ✅ **YAML configuration**: Resource definitions
- ✅ **Service discovery**: How applications find each other

**Ready for Phase 2: Application Deployment? Let's learn about Deployments, ConfigMaps, and Health Checks! 🚀** 