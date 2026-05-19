# Cloud-Native Todo Microservices Application

Production-ready cloud-native Todo application built with Kubernetes, GitOps workflows, Kustomize overlays, and Helm packaging.

This project demonstrates the evolution of a Kubernetes application lifecycle:

**Raw manifests → Kustomize environments → GitOps overlays → Helm chart packaging → NetworkPolicy isolation**

---

# 📸 Application Preview

## Infrastructure Status

![Minikube Cluster Status](docs/screenshots/minikube-status.png)

## Application UI

![Application Frontend UI](docs/screenshots/app-frontend.png)

---

# 🏗 Architecture

Traffic is routed through a Traefik Ingress Controller and distributed between multiple microservices.

```text
                           [ Traefik Ingress ]
                             (todolist.local)
                                      |
         +----------------------------+---------------------------+
         | /                          | /webui                    | /todos
         v                            v                           v

 [ Vue.js Frontend ]         [ Flask Frontend ]          [ REST API Backend ]
     Port 8080                  Port 5000                    Port 8080

         |                            |                           |
         +-------------+--------------+                           |
                       |                                          |
                       v                                          v

                  [ CoreDNS ]                         [ MariaDB StatefulSet ]
                   Port 53                           mariadb-headless:3306
```

---

# 📦 Project Structure

```text
.
├── charts/                 # Helm chart package
├── kustomize/
│   ├── base/
│   └── overlays/
│       ├── stateful/
│       ├── stateful-hpa/
│       ├── stateful-hpa-probes/
│       └── production/
├── manifests/
│   └── crds/
└── docs/
    └── screenshots/
```

---

# ⚙ Prerequisites

Before deployment, ensure the following tools are installed:

- Kubernetes cluster (**Minikube** recommended)
- Helm v3+
- kubectl
- Traefik / Ingress addon enabled

Enable ingress:

```bash
minikube addons enable ingress
```

---

# 🚀 Deployment Guide

## Step 1 — Configure Local Routing

Start the Minikube tunnel:

```bash
minikube tunnel
```

Add the following entry to your hosts file:

Linux / macOS:

```bash
sudo nano /etc/hosts
```

Windows:

```text
C:\Windows\System32\drivers\etc\hosts
```

Append:

```text
127.0.0.1 todolist.local
```

---

## Step 2 — Deploy the Application

### Option A — Helm Deployment (Recommended)

Deploy directly from the OCI registry:

```bash
helm install todolist-test oci://ghcr.io/matang477/todolist \
    --version 0.1.0 \
    --set mysql.rootPassword=Test1234 \
    --set ingress.host=todolist.local
```

---

### Option B — Kustomize Deployment

Deploy manually using overlays:

Apply base resources:

```bash
kubectl apply -k kustomize/base/
```

Deploy production overlay:

```bash
kubectl apply -k kustomize/overlays/production/
```

---

## Step 3 — Access the Application

Wait until all pods reach **Running** status:

```bash
kubectl get pods
```

Then open:

| Service | URL |
|----------|------|
| Vue Frontend | http://todolist.local/ |
| Flask UI | http://todolist.local/webui |
| REST API | http://todolist.local/todos |

---

# 🛠 Infrastructure Layers

## 1. Kustomize Overlay Pipeline

### `base/`

Core resources:

- Deployments
- Services
- Persistent volumes
- Backup jobs

### `overlays/stateful/`

Migrates MariaDB from Deployment → StatefulSet.

Includes:

- Headless service
- Persistent storage
- Ordered startup behavior

### `overlays/stateful-hpa/`

Adds autoscaling:

- HorizontalPodAutoscaler
- Metrics-based scaling

### `overlays/stateful-hpa-probes/`

Adds health checks:

- `/livez`
- `/readyz`

### `overlays/production/`

Production environment configuration:

- Ingress
- ConfigMaps
- NetworkPolicies
- Path routing

---

## 2. Zero-Trust Networking

Network isolation enforced via Kubernetes NetworkPolicies.

### Database Tier

Allowed:

- Traffic only from application services
- Port `3306`

Blocked:

- External access
- Lateral pod traffic

### Application Tier

Restricted egress:

- MariaDB
- CoreDNS (`53`)

---

## 3. Helm Packaging

The entire application stack is packaged as a parameterized Helm v3 chart:

```text
charts/
```

Published through OCI registry:

```text
ghcr.io/matang477/todolist
```

---

## 4. Kubernetes API Extensions

Custom Kubernetes resources located under:

```text
manifests/crds/
```

Provides custom resource definitions extending native Kubernetes APIs.

---

# 🔧 Technologies Used

- Kubernetes
- Minikube
- Helm v3
- Kustomize
- Traefik
- MariaDB
- Flask
- Vue.js
- NetworkPolicies
- GitOps workflows
