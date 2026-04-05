# 🐳 Docker & ☸️ Kubernetes — Complete Notes
> From Fresher to Experienced Level

---

## Table of Contents

1. [Docker Overview](#1-docker-overview)
2. [How Docker Works](#2-how-docker-works)
3. [Docker Core Concepts (Deep Dive)](#3-docker-core-concepts-deep-dive)
4. [Kubernetes Overview](#4-kubernetes-overview)
5. [Kubernetes Architecture](#5-kubernetes-architecture)
6. [Kubernetes Cluster Setup](#6-kubernetes-cluster-setup)
7. [MiniKube Setup (Step-by-Step)](#7-minikube-setup-step-by-step)
8. [Pods in Kubernetes](#8-pods-in-kubernetes)
9. [Kubernetes Services](#9-kubernetes-services)
10. [Manifest YAML Files](#10-manifest-yaml-files)
11. [Namespaces](#11-namespaces)
12. [Kubernetes Resources & Controllers](#12-kubernetes-resources--controllers)
13. [Deployments](#13-deployments)
14. [Scaling in Kubernetes](#14-scaling-in-kubernetes)
15. [CI/CD Pipeline with Jenkins + Docker + K8s](#15-cicd-pipeline-with-jenkins--docker--k8s)
16. [EKS (AWS Kubernetes)](#16-eks-aws-kubernetes)
17. [Quick Reference — Commands Cheat Sheet](#17-quick-reference--commands-cheat-sheet)

---

## 1. Docker Overview

### What is Docker?

Docker is a **free and open-source containerization platform** that packages applications along with all their dependencies into a single portable unit called a **Docker Image**.

> 💡 Think of Docker like a shipping container — it packages everything your app needs so it runs the same way everywhere.

### Why Use Docker?

| Feature | Benefit |
|---|---|
| ✅ Portability | Run the same app on any machine regardless of OS |
| ✅ Dependency Management | All required libraries, runtimes, DBs are included |
| ✅ Fast Deployment | No manual setup needed on new environments |
| ✅ Resource Efficiency | Uses far fewer resources than traditional VMs |
| ✅ Isolation | Each container runs in its own isolated environment |
| ✅ Version Control | Images can be versioned and rolled back |

### Docker vs Virtual Machine (VM)

| Feature | Docker Container | Virtual Machine |
|---|---|---|
| Boot Time | Seconds | Minutes |
| Size | MBs | GBs |
| OS | Shares host OS kernel | Full OS per VM |
| Performance | Near-native | Overhead from hypervisor |
| Isolation | Process-level | Full hardware-level |

> 🧠 **Freshers:** Containers share the host OS kernel. VMs have their own OS. This makes containers much lighter and faster.

---

## 2. How Docker Works

```
Developer Code + Dependencies
          ↓
    [ Dockerfile ]
          ↓
    [ Docker Image ]   ← Immutable snapshot
          ↓
    [ Docker Container ] ← Running instance of the image
```

### Step-by-Step Flow

1. **Write a Dockerfile** — Instructions to build the image (what OS, what software, copy code, run command).
2. **Build a Docker Image** — `docker build -t myapp:latest .`
3. **Run as Container** — `docker run -p 8080:8080 myapp:latest`
4. **Push to Registry** — `docker push myapp:latest` (e.g., Docker Hub, ECR)
5. **Pull & Run Anywhere** — Any machine with Docker installed can pull and run this image.

### Sample Dockerfile

```dockerfile
# Base image
FROM openjdk:17-jdk-slim

# Set working directory inside container
WORKDIR /app

# Copy built jar file
COPY target/myapp.jar app.jar

# Expose port
EXPOSE 9090

# Command to run the app
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Common Docker Commands

```bash
# Build image
docker build -t myapp:latest .

# Run container
docker run -d -p 8080:9090 --name mycontainer myapp:latest

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop container
docker stop mycontainer

# Remove container
docker rm mycontainer

# List images
docker images

# Remove image
docker rmi myapp:latest

# View container logs
docker logs mycontainer

# Execute command inside container
docker exec -it mycontainer bash

# Pull image from Docker Hub
docker pull nginx:latest

# Push image to Docker Hub
docker push username/myapp:latest
```

---

## 3. Docker Core Concepts (Deep Dive)

### Docker Image vs Container

| Docker Image | Docker Container |
|---|---|
| Blueprint / Template | Running instance of image |
| Read-only | Has a writable layer on top |
| Stored in registry | Lives in memory/disk at runtime |
| `docker build` creates it | `docker run` creates it |

### Docker Registry

A **registry** is where Docker images are stored and shared.

- **Docker Hub** — Public registry (hub.docker.com)
- **AWS ECR** — Amazon Elastic Container Registry
- **GCP GCR** — Google Container Registry
- **Azure ACR** — Azure Container Registry
- **Private Registry** — Self-hosted using `registry` image

### Docker Networking (Brief)

| Type | Description |
|---|---|
| `bridge` | Default. Containers communicate via internal bridge network |
| `host` | Container uses host machine's network directly |
| `none` | No networking |
| `overlay` | Multi-host networking (used in Swarm/K8s) |

### Docker Volumes (Persistent Storage)

Containers are **ephemeral** — data is lost when a container is removed. Use volumes for persistence.

```bash
# Create a volume
docker volume create mydata

# Mount volume when running container
docker run -v mydata:/app/data myapp:latest

# Bind mount (map host directory)
docker run -v /host/path:/container/path myapp:latest
```

---

## 4. Kubernetes Overview

### What is Kubernetes?

**Kubernetes (K8s)** is a free, open-source **container orchestration platform** developed by Google, now maintained by the CNCF (Cloud Native Computing Foundation).

> 💡 **Orchestration** = Managing many containers automatically — starting, stopping, scaling, healing, and networking them.

### Why Kubernetes?

| Feature | Description |
|---|---|
| ✅ Orchestration | Manages 100s/1000s of containers across many machines |
| ✅ Self-Healing | Automatically restarts failed containers |
| ✅ Load Balancing | Distributes traffic evenly across Pods |
| ✅ Auto Scaling | Scales up/down based on demand |
| ✅ Automated Deployments | Rolling updates with zero downtime |
| ✅ Storage Orchestration | Automatically mounts storage from cloud or local |
| ✅ Config Management | Manages secrets and configs securely |

### Docker vs Kubernetes

| Docker | Kubernetes |
|---|---|
| Runs a single container | Manages many containers across many nodes |
| Manual scaling | Automatic scaling |
| No self-healing | Auto-restarts failed containers |
| Basic networking | Advanced networking and service discovery |
| Good for dev/testing | Production-grade orchestration |

> 🧠 **Key Insight:** Docker and Kubernetes are complementary, not competing. Docker builds and runs containers; Kubernetes orchestrates them.

---

## 5. Kubernetes Architecture

```
┌─────────────────────────────────────────────┐
│             CONTROL PLANE (Master)           │
│                                              │
│  ┌──────────┐  ┌───────────┐  ┌──────────┐  │
│  │API Server│  │ Scheduler │  │Controller│  │
│  │          │  │           │  │ Manager  │  │
│  └──────────┘  └───────────┘  └──────────┘  │
│                    ┌──────┐                  │
│                    │ ETCD │                  │
│                    └──────┘                  │
└─────────────────────────────────────────────┘
           ↕ (API calls via kubectl)
┌───────────────────┐   ┌───────────────────┐
│   Worker Node 1   │   │   Worker Node 2   │
│                   │   │                   │
│ ┌───────────────┐ │   │ ┌───────────────┐ │
│ │    Kubelet    │ │   │ │    Kubelet    │ │
│ ├───────────────┤ │   │ ├───────────────┤ │
│ │  Kube Proxy   │ │   │ │  Kube Proxy   │ │
│ ├───────────────┤ │   │ ├───────────────┤ │
│ │ Docker Engine │ │   │ │ Docker Engine │ │
│ ├───────────────┤ │   │ ├───────────────┤ │
│ │  Pod  │  Pod  │ │   │ │  Pod  │  Pod  │ │
│ └───────────────┘ │   │ └───────────────┘ │
└───────────────────┘   └───────────────────┘
```

### Control Plane Components

| Component | Role |
|---|---|
| **API Server** | The gateway — receives all requests from `kubectl` and other clients. Every operation goes through here. |
| **Scheduler** | Decides *which* worker node a new Pod should run on (based on resources, constraints, etc.) |
| **Controller Manager** | Continuously monitors the cluster to ensure the *desired state* matches the *actual state* |
| **ETCD** | Distributed key-value store — the "brain" / database of the cluster. Stores all cluster state and config. |

> 🧠 **ETCD** is the source of truth. If ETCD goes down, the cluster loses its memory. Always back it up in production.

### Worker Node Components

| Component | Role |
|---|---|
| **Kubelet** | Node agent that talks to the Control Plane. Ensures containers are running in Pods as instructed. |
| **Kube Proxy** | Handles all networking within the cluster. Routes traffic between Pods and Services. |
| **Docker Engine** | (or containerd) — The container runtime that actually runs containers. |
| **Pod** | Smallest deployable unit — wraps one or more containers. |
| **Container** | The actual running application inside a Pod. |

### How a Deployment Request Works (Step-by-Step)

```
You (kubectl apply)
     ↓
API Server  → stores request in ETCD with "Pending" status
     ↓
Scheduler   → finds a suitable worker node
     ↓
Kubelet     → receives the assignment, pulls image, starts container
     ↓
Kube Proxy  → sets up networking rules
     ↓
Controller Manager → continuously monitors, ensures desired replicas are running
```

---

## 6. Kubernetes Cluster Setup

A **Kubernetes Cluster** = Control Plane + Worker Nodes + Networking + Storage

### Option 1: Self-Managed (On-Premise)

| Tool | Use Case |
|---|---|
| **MiniKube** | Single-node local cluster — ideal for learning |
| **Kubeadm** | Multi-node cluster — full control, requires expertise |
| **Kind** (K8s in Docker) | Lightweight multi-node cluster for testing |
| **k3s** | Lightweight K8s for IoT/edge environments |

### Option 2: Cloud-Managed (Production)

| Cloud | Service | Notes |
|---|---|---|
| AWS | EKS (Elastic Kubernetes Service) | Most widely used in enterprises |
| Azure | AKS (Azure Kubernetes Service) | Deep Azure integration |
| GCP | GKE (Google Kubernetes Engine) | Most mature, Google-native |
| DigitalOcean | DOKS | Good for smaller teams |

> ✅ **Recommendation for Production:** Always use a cloud-managed cluster (EKS/AKS/GKE). They handle control plane availability, patches, and upgrades for you.

---

## 7. MiniKube Setup (Step-by-Step)

> Prerequisites: AWS Ubuntu VM — t2.medium, 50GB storage, 2 vCPU

### Step 1: Create Ubuntu VM on AWS

```
- Login to AWS Console
- EC2 → Launch Instance
- AMI: Ubuntu 22.04 LTS
- Instance Type: t2.medium (2 vCPU, 4GB RAM)
- Storage: 50 GB
- Security Group: Allow SSH (port 22), HTTP (80), NodePort range (30000-32767)
- Launch and connect via SSH
```

### Step 2: Install Docker

```bash
sudo apt update
curl -fsSL get.docker.com | /bin/bash
sudo usermod -aG docker ubuntu
exit  # Log out and back in for group change to take effect
```

### Step 3: Install MiniKube Dependencies

```bash
sudo apt update
sudo apt install -y curl wget apt-transport-https
```

### Step 4: Install MiniKube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version  # Verify installation
```

### Step 5: Install kubectl

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s \
  https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version -o yaml  # Verify installation
```

### Step 6: Start MiniKube

```bash
minikube start --driver=docker
```

### Step 7: Verify Setup

```bash
minikube status        # Check MiniKube status
kubectl cluster-info   # View cluster info
kubectl get nodes      # View worker nodes
```

> ⚠️ **Note:** MiniKube is **not suitable for production**. It's single-node, has no high availability, and is meant only for local development and learning.

---

## 8. Pods in Kubernetes

### What is a Pod?

A **Pod** is the **smallest deployable unit** in Kubernetes. It wraps one or more containers.

```
Pod
 └── Container 1 (your app)
 └── Container 2 (optional - sidecar, like a log collector)
```

> 💡 **Analogy:** A Pod is like a "host" for your container. Just as a Docker container wraps your app, a Pod wraps the container and adds K8s-level management.

### Key Facts About Pods

- Every Pod gets its own **IP address** within the cluster.
- Pods are **ephemeral** — they can die and be replaced at any time.
- **Never directly rely on a Pod's IP** — use a Service instead.
- A Pod can run **multiple containers** that share the same network and storage.
- Pods are **defined using YAML manifest files**.

### Multi-Pod Benefits

```
            [Load Balancer / Service]
           /          |           \
        Pod 1       Pod 2       Pod 3
     (App v1.0)  (App v1.0)  (App v1.0)
```

- **High Availability:** If Pod 1 crashes, Pod 2 and 3 still serve requests.
- **Load Balancing:** Traffic is distributed across all healthy Pods.
- **Scalability:** Add more Pods during peak traffic, remove during off-peak.

### Pod Lifecycle

```
Pending → Running → Succeeded / Failed / Unknown
```

| Status | Meaning |
|---|---|
| Pending | Pod is scheduled but container not yet started |
| Running | Container is running |
| Succeeded | All containers exited successfully (jobs) |
| Failed | Container exited with error |
| Unknown | Node communication error |

---

## 9. Kubernetes Services

### Why Services?

Pods are **temporary** — they crash, restart, and get new IP addresses each time. A **Service** provides a **stable network identity** (fixed IP and DNS name) for a group of Pods.

```
Client → Service (stable IP) → Pod 1
                              → Pod 2
                              → Pod 3
```

### Types of Services

#### 🔐 ClusterIP (Default — Internal Only)

```yaml
spec:
  type: ClusterIP
```

- Accessible **only within the cluster**.
- Other Pods inside the cluster can reach it.
- **Use for:** Databases, internal APIs, backend microservices.

```
[Frontend Pod] → ClusterIP Service → [Backend/DB Pods]
(inside cluster only)
```

#### 🌐 NodePort (External via Node IP)

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 9090
      nodePort: 30080    # Range: 30000–32767
```

- Opens a port on **every worker node**.
- Accessible via `http://<NodeIP>:<NodePort>`.
- Traffic goes to **one specific node** — no real load balancing across nodes.
- **Use for:** Dev/testing, simple external access.

#### ⚖️ LoadBalancer (Production External Access)

```yaml
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 9090
```

- Creates a **cloud load balancer** (AWS ELB, Azure LB, GCP LB).
- Automatically distributes traffic across all Pods.
- Gets an **external IP/DNS** from the cloud provider.
- **Use for:** Production-grade public-facing applications.

#### 🔎 ExternalName (Advanced)

Maps a Service to an external DNS name. Used for accessing external services inside the cluster using a K8s Service name.

### Service → Pod Routing (How It Works)

Services use **label selectors** to find Pods. The Service watches for Pods with matching labels and routes traffic to them.

```yaml
# Pod has label:
labels:
  app: dempapp

# Service targets pods with that label:
selector:
  app: dempapp
```

---

## 10. Manifest YAML Files

### Structure of Any Kubernetes Manifest

```yaml
apiVersion: <version>     # Which K8s API to use
kind: <resource-type>     # Pod, Service, Deployment, etc.
metadata:                 # Name, namespace, labels
  name: my-resource
  labels:
    app: myapp
spec:                     # The desired state / configuration
  ...
```

### Pod Manifest Example

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: testpod
  labels:
    app: dempapp
spec:
  containers:
    - name: test
      image: psait/pankajsiracademy:latest
      ports:
        - containerPort: 9090
      # Optional: Resource limits (best practice)
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
```

### Service Manifest Example (NodePort)

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: testpod-service
spec:
  type: NodePort
  selector:
    app: dempapp           # Must match Pod's label
  ports:
    - port: 80             # Port the Service listens on internally
      targetPort: 9090     # Port the container app listens on
      nodePort: 30080      # Port exposed on each node (30000–32767)
```

### Essential kubectl Commands

```bash
# Apply a manifest
kubectl apply -f <filename>.yml

# Delete resources defined in a manifest
kubectl delete -f <filename>.yml

# Check pods
kubectl get pods
kubectl get pods -o wide    # More details (node, IP)

# Check pod logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow/stream logs

# Describe a pod (detailed info + events)
kubectl describe pod <pod-name>

# Execute into a running pod
kubectl exec -it <pod-name> -- bash

# Check all resources
kubectl get all

# Delete all resources
kubectl delete all --all

# Delete specific resources
kubectl delete pod <pod-name>
kubectl delete svc <service-name>
```

---

## 11. Namespaces

### What are Namespaces?

**Namespaces** logically divide a Kubernetes cluster into virtual sub-clusters. Think of them like folders on a computer — they isolate resources.

```
Kubernetes Cluster
├── default              ← Resources created without specifying a namespace
├── kube-system          ← K8s internal components (DNS, dashboard, etc.)
├── kube-public          ← Publicly readable resources
└── [custom namespaces]
    ├── backend-ns       ← Backend services
    ├── frontend-ns      ← Frontend services
    └── database-ns      ← Databases
```

### Why Use Namespaces?

- **Team Isolation:** Team A can't accidentally delete Team B's resources.
- **Environment Isolation:** `dev`, `staging`, `prod` in one cluster.
- **Resource Quotas:** Limit CPU/memory per namespace.
- **Access Control:** RBAC policies per namespace.

### Namespace Commands

```bash
# List all namespaces
kubectl get ns

# Get resources in a specific namespace
kubectl get pods -n backend-ns
kubectl get all -n backend-ns

# Get pods in kube-system (internal K8s pods)
kubectl get pods -n kube-system
```

### Create Namespace (Two Ways)

**CLI:**
```bash
kubectl create namespace backend-ns
```

**YAML:**
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: backend-ns
```

### Full Example: Namespace + Pod + Service

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: backend-ns
---
apiVersion: v1
kind: Pod
metadata:
  name: testpod
  namespace: backend-ns      # Assign to namespace
  labels:
    app: dempapp
spec:
  containers:
    - name: test
      image: psait/pankajsiracademy:latest
      ports:
        - containerPort: 9090
---
apiVersion: v1
kind: Service
metadata:
  name: testpod-service
  namespace: backend-ns      # Must be in the same namespace as the Pod
spec:
  type: NodePort
  selector:
    app: dempapp
  ports:
    - port: 80
      targetPort: 9090
      nodePort: 30080
```

```bash
# Delete namespace (deletes ALL resources inside it)
kubectl delete ns backend-ns
```

> ⚠️ **Warning:** Deleting a namespace deletes all resources inside it. Be careful in production!

---

## 12. Kubernetes Resources & Controllers

### The Problem with Bare Pods

When you create a Pod directly (`kind: Pod`), Kubernetes **does not manage its lifecycle**. If it crashes or gets deleted — it's gone forever.

```
Bare Pod crashes → Pod is GONE forever ❌
Deployment Pod crashes → K8s creates a new one automatically ✅
```

### Controllers Overview

Controllers are K8s resources that **manage Pod lifecycle** — ensuring the right number of Pods are always running.

| Controller | Use Case |
|---|---|
| **ReplicationController (RC)** | Old way. Keeps N pods running. |
| **ReplicaSet (RS)** | Modern RC. Supports more flexible label selectors. |
| **Deployment** | Most commonly used. Adds rolling updates & rollbacks on top of RS. |
| **DaemonSet** | Runs one Pod per node (e.g., log collector, monitoring agent). |
| **StatefulSet** | For stateful apps like databases (MySQL, Kafka) — stable identity. |
| **Job** | Runs a Pod to completion (batch processing). |
| **CronJob** | Scheduled Jobs (like cron). |

---

### ReplicationController (RC)

```yaml
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: webapp
spec:
  replicas: 3                  # Always keep 3 pods running
  selector:
    app: dempapp
  template:
    metadata:
      labels:
        app: dempapp
    spec:
      containers:
        - name: webappcontainer
          image: psait/pankajsiracademy:latest
          ports:
            - containerPort: 9090
```

```bash
kubectl apply -f rc.yml
kubectl get rc
kubectl scale rc webapp --replicas=5    # Scale up
kubectl scale rc webapp --replicas=1    # Scale down
```

> ⚠️ ReplicationController is largely **replaced by ReplicaSet**. Use Deployments in practice.

---

### ReplicaSet (RS)

ReplicaSet is the **modern replacement for RC**. The key difference is a more flexible `matchExpressions` selector.

```yaml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dempapp             # Manages Pods with this label
  template:
    metadata:
      labels:
        app: dempapp
    spec:
      containers:
        - name: webappcontainer
          image: psait/pankajsiracademy:latest
          ports:
            - containerPort: 9090
```

> 💡 ReplicaSet can also **adopt pre-existing Pods** that match its selector — even pods created manually before the RS was created.

```bash
kubectl get rs
kubectl scale rs webapp --replicas=5
```

> 🧠 In real-world projects, **you almost never write a ReplicaSet directly**. You write a Deployment, which creates and manages a ReplicaSet for you.

---

## 13. Deployments

### What is a Deployment?

A **Deployment** is the most widely used Kubernetes resource for managing stateless applications. It creates and manages a ReplicaSet under the hood, adding:

- ✅ **Rolling Updates** — Update Pods gradually with zero downtime.
- ✅ **Rollback** — Instantly revert to a previous version.
- ✅ **Pause/Resume** — Pause a rollout mid-way.

### Deployment vs ReplicaSet

| Feature | ReplicaSet | Deployment |
|---|---|---|
| Manages Pods | ✅ | ✅ (via RS) |
| Rolling Updates | ❌ | ✅ |
| Rollback | ❌ | ✅ |
| Pause/Resume | ❌ | ✅ |
| Real-world use | Rare | Very common |

### Deployment Manifest

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate            # Or: Recreate
    rollingUpdate:
      maxSurge: 1                  # How many extra pods allowed during update
      maxUnavailable: 1            # How many pods can be down during update
  selector:
    matchLabels:
      app: dempapp
  template:
    metadata:
      labels:
        app: dempapp
    spec:
      containers:
        - name: webappcontainer
          image: psait/pankajsiracademy:latest
          ports:
            - containerPort: 9090
---
apiVersion: v1
kind: Service
metadata:
  name: webappservice
spec:
  type: NodePort
  selector:
    app: dempapp
  ports:
    - port: 80
      targetPort: 9090
      nodePort: 30095
```

### Update Strategies

#### RollingUpdate (Default — Recommended for Production)

```
Old Pods:  [v1] [v1] [v1]
Step 1:    [v1] [v1] [v2]    ← 1 new pod started
Step 2:    [v1] [v2] [v2]    ← 1 more updated
Step 3:    [v2] [v2] [v2]    ← All updated, zero downtime
```

- Gradually replaces old Pods with new ones.
- App stays available throughout the update.
- Use for most production scenarios.

#### Recreate (For Maintenance Windows)

```
Old Pods:  [v1] [v1] [v1]
Step 1:    [ ]  [ ]  [ ]     ← All old pods terminated
Step 2:    [v2] [v2] [v2]    ← All new pods started
```

- All old Pods killed, then all new Pods started.
- Results in **brief downtime**.
- Use when you can't run old and new versions simultaneously.

### Deployment Commands

```bash
# Apply deployment
kubectl apply -f deployment.yml

# Check deployments
kubectl get deployments
kubectl get deploy

# Check rollout status
kubectl rollout status deployment/webapp

# View rollout history
kubectl rollout history deployment/webapp

# Rollback to previous version
kubectl rollout undo deployment/webapp

# Rollback to specific revision
kubectl rollout undo deployment/webapp --to-revision=2

# Update image (triggers rolling update)
kubectl set image deployment/webapp webappcontainer=psait/pankajsiracademy:v2

# Scale deployment
kubectl scale deployment webapp --replicas=5

# Pause rollout
kubectl rollout pause deployment/webapp

# Resume rollout
kubectl rollout resume deployment/webapp
```

---

## 14. Scaling in Kubernetes

### Types of Scaling

| Type | Description |
|---|---|
| **Manual Scaling** | You manually change replica count |
| **HPA (Horizontal Pod Autoscaler)** | Automatically adds/removes Pods based on CPU/memory |
| **VPA (Vertical Pod Autoscaler)** | Automatically adjusts CPU/memory requests per Pod |
| **Cluster Autoscaler** | Automatically adds/removes worker nodes |

### HPA — Horizontal Pod Autoscaler

HPA watches metrics (CPU, memory, custom) and scales the number of Pods up or down automatically.

```
Traffic increases → CPU goes above 80% → HPA adds more Pods
Traffic decreases → CPU falls below 20% → HPA removes Pods
```

#### HPA Manifest

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70   # Scale when CPU > 70%
```

> ⚠️ **Prerequisite:** HPA requires **Metrics Server** to be running in the cluster. Install with:
> ```bash
> kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
> ```

#### HPA Commands

```bash
# Get HPA status
kubectl get hpa
kubectl get hpa -w      # Watch in real time

# Describe HPA
kubectl describe hpa webapp-hpa
```

#### Load Testing HPA

```bash
# Generate load to trigger HPA scaling
kubectl run -i --tty load-generator --rm \
  --image=busybox --restart=Never \
  -- /bin/sh -c "while true; do wget -q -O- http://webappservice; sleep 0.01; done"
```

### VPA — Vertical Pod Autoscaler

Instead of adding more Pods, VPA **increases or decreases the CPU/memory resources** for existing Pods.

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: webapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  updatePolicy:
    updateMode: "Auto"      # Auto, Off, or Initial
```

> 💡 **When to use HPA vs VPA?**
> - Use **HPA** for stateless apps (web servers, APIs) — just add more Pods.
> - Use **VPA** for stateful apps or when horizontal scaling isn't possible.

---

## 15. CI/CD Pipeline with Jenkins + Docker + K8s

### Pipeline Flow

```
Code Push (GitHub)
     ↓
Jenkins Pipeline
  Stage 1: Clone Repo       ← git clone
  Stage 2: Maven Build      ← mvn clean package
  Stage 3: Docker Image     ← docker build + push
  Stage 4: K8s Deploy       ← kubectl apply
     ↓
Kubernetes Cluster (Updated Pods Running)
```

### Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    tools {
        maven "maven-3.9.9"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/yourorg/your-repo.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Image Build & Push') {
            steps {
                sh 'docker build -t psait/pankajsiracademy:latest .'
                // Optional: push to registry
                // sh 'docker push psait/pankajsiracademy:latest'
            }
        }
        stage('K8s Deployment') {
            steps {
                sh 'kubectl apply -f k8s-deploy.yml'
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Build/Deploy Failed. Check logs.'
        }
    }
}
```

> 💡 For production pipelines, also add:
> - Image tagging with build number (`:${BUILD_NUMBER}`)
> - Docker registry credentials using Jenkins secrets
> - Rollback stage on failure
> - Notification (Slack/Email) on success/failure

---

## 16. EKS (AWS Kubernetes)

### What is EKS?

**Amazon Elastic Kubernetes Service (EKS)** is AWS's fully managed Kubernetes service. AWS manages the Control Plane; you manage Worker Nodes (EC2 instances).

### EKS Architecture

```
AWS Account
├── EKS Control Plane (Managed by AWS)
│   ├── API Server
│   ├── ETCD
│   └── Controllers
└── Worker Nodes (EC2 instances — you manage)
    ├── Node Group 1 (t2.medium × 2)
    └── Node Group 2 (t2.medium × 2)
```

### Create EKS Cluster with eksctl

```bash
# Install eksctl (one-time setup)
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create cluster
eksctl create cluster \
  --name my-cluster \
  --region ap-south-1 \
  --node-type t2.medium \
  --zones ap-south-1a,ap-south-1b \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 4

# Verify cluster
kubectl get nodes

# Delete cluster
eksctl delete cluster --name my-cluster
```

### EKS Cluster via YAML Config

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: ap-south-1

availabilityZones: ["ap-south-1a", "ap-south-1b"]

nodeGroups:
  - name: ng-1
    instanceType: t2.medium
    desiredCapacity: 2
    minSize: 2
    maxSize: 6
    volumeSize: 20
```

```bash
eksctl create cluster -f cluster.yaml
```

### Deploying App to EKS with LoadBalancer

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata:
      labels:
        app: javawebapp
    spec:
      containers:
        - name: webappcontainer
          image: psait/pankajsiracademy:latest
          ports:
            - containerPort: 9090
---
apiVersion: v1
kind: Service
metadata:
  name: websvc
spec:
  type: LoadBalancer       # AWS will provision an ELB automatically
  selector:
    app: javawebapp
  ports:
    - port: 80
      targetPort: 9090
```

> 💡 When `type: LoadBalancer` is used on EKS, AWS automatically creates an **Elastic Load Balancer (ELB)** and assigns it a public DNS. You use that DNS to access your app.

---

## 17. Quick Reference — Commands Cheat Sheet

### MiniKube

```bash
minikube start --driver=docker    # Start cluster
minikube stop                     # Stop cluster
minikube delete                   # Delete cluster
minikube status                   # Check status
minikube ip                       # Get MiniKube IP
minikube service <svc-name>       # Open service in browser
```

### kubectl — Cluster Info

```bash
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl config view
kubectl config current-context
```

### kubectl — Pods

```bash
kubectl get pods
kubectl get pods -n <namespace>
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs -f <pod-name>                    # Stream logs
kubectl exec -it <pod-name> -- bash          # Shell into pod
kubectl delete pod <pod-name>
```

### kubectl — Deployments

```bash
kubectl get deployments
kubectl describe deployment <name>
kubectl scale deployment <name> --replicas=5
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl set image deployment/<name> <container>=<new-image>
```

### kubectl — Services

```bash
kubectl get svc
kubectl get svc -n <namespace>
kubectl describe svc <service-name>
kubectl delete svc <service-name>
```

### kubectl — Namespaces

```bash
kubectl get ns
kubectl create namespace <name>
kubectl get all -n <namespace>
kubectl delete ns <namespace>
```

### kubectl — Apply & Manage

```bash
kubectl apply -f <file>.yml          # Create or update resource
kubectl delete -f <file>.yml         # Delete resource from file
kubectl delete all --all             # Delete all resources in current namespace
kubectl get all                      # View all resources
kubectl get all -n <namespace>       # View all in a namespace
```

### kubectl — Debugging

```bash
kubectl describe pod <pod-name>            # Full details + events
kubectl logs <pod-name>                    # Container logs
kubectl logs <pod-name> -c <container>    # Specific container in multi-container pod
kubectl get events                         # Cluster events
kubectl top pods                           # CPU/memory usage (needs metrics-server)
kubectl top nodes
```

---

## Summary

```
Docker                      Kubernetes
──────────────────────────────────────────────────────────
Build image (Dockerfile)    Deploy using Pods
Run container locally       Manage with Deployments
Push to registry            Auto-scale with HPA
                            Expose with Services
                            Isolate with Namespaces
                            Orchestrate with K8s Controllers
```

> 🚀 **Docker** makes apps portable by containerizing them.
> 🚀 **Kubernetes** makes containerized apps **production-ready** — scalable, resilient, and self-healing.

Together, they form the backbone of modern **cloud-native application delivery**.

---

*Notes compiled for learning purposes — Fresher to Experienced Level*
