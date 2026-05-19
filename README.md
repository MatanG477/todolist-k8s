# Cloud-Native Microservices Todo Application Stack

A production-ready microservices application orchestrated with Kubernetes. This project traces a comprehensive engineering lifecycle—evolving from baseline declarative specifications into structured, multi-environment GitOps Kustomize pipelines, and ultimately abstracting the entire lifecycle into an enterprise-grade Helm v3 package with zero-trust network topology.

---

## 📸 Application Preview

### Core Infrastructure State
![Minikube Cluster Status](docs/screenshots/minikube-status.png)

### Live Application Workspace
![Application Frontend UI](docs/screenshots/app-frontend.png)

---

## 🏗️ System Architecture & Traffic Routing

Traffic enters via a unified ingress abstraction layer, dynamically routing web traffic and state operations across decoupled microservice runtimes using a Traefik Ingress Controller:

```text
                           [ Traefik Ingress ] (todolist.local)
                                    |
         +--------------------------+--------------------------+
         | /                        | /webui                   | /todos
         v                          v                          v
 [ Vue.js Web UI ]         [ Flask Frontend ]          [ REST API Backend ]
   (Port 8080)               (Port 5000)                 (Port 8080)
         |                          |                          |
         +-------------+------------+                          |
                       | (Egress Allowed)                      | (Egress Allowed)
                       v                                       v
               [ CoreDNS (Port 53) ]               [ MariaDB StatefulSet ]
                                                       (mariadb-headless:3306)
```                                                       

📥 STEP-BY-STEP INSTALLATION & RUN GUIDE

Follow these exact steps to cleanly build and execute the microservices environment locally on your machine.
📋 Prerequisites

Ensure your local terminal profile has the following tools active before beginning:

    Kubernetes Cluster Environment: Minikube Installed & Running

    Package Manager: Helm v3+ Installed

    Ingress Addon: Enable the Traefik routing controller on your active cluster:
    Bash

    minikube addons enable ingress

Step 1: Configure Local DNS Network Routing

Because this application runs path-based routing rules over a unified virtual domain name (todolist.local), you must map local traffic to your cluster:

    Launch a persistent load-balancer routing tunnel in a separate terminal window (leave this running):
    Bash

    minikube tunnel

    Open your system hosts configuration file (/etc/hosts on Linux/macOS or C:\Windows\System32\drivers\etc\hosts on Windows) and append this domain routing rule to the bottom:
    Plaintext

    127.0.0.1 todolist.local

Step 2: Deploy the Application Components
Method A: Direct Automation via OCI Helm Registry (Recommended)

To stand up the complete cluster structure instantly with an active connection to the published package repository registry, execute this command block:
Bash

helm install todolist-test oci://ghcr.io/matang477/todolist \
  --version 0.1.0 \
  --set mysql.rootPassword=Test1234 \
  --set ingress.host=todolist.local

Method B: Manual GitOps Overlay Rollout (Alternative)

If you prefer to deploy using the progressive Kustomize manifest pipeline layers manually instead of Helm, run these standard kubectl commands from the root of the repository:
Bash

# Apply baseline primitives
kubectl apply -k kustomize/base/

# Patch up to the complete production specification layer
kubectl apply -k kustomize/overlays/production/

Step 3: Access the Live Application Endpoints

Give the cluster 20–30 seconds to launch all of your application containers. Once your pods show a status of Running, open your web browser to navigate directly to the isolated microservice nodes:

    Modern Vue.js Client Web Workspace: http://todolist.local/

    Legacy Flask UI Environment: http://todolist.local/webui

    Core REST Backend API Engine: http://todolist.local/todos

🛠️ How the Application is Built

This cloud-native application is engineered using advanced Kubernetes infrastructure standards across three core development tracking layers:
1. Progressive Kustomize Overlay Framework (/kustomize)

Instead of maintaining loose, hardcoded duplicate YAML files for multiple runtime environments, this layout builds a modular GitOps patching pipeline:

    base/: Core workload baselines establishing basic microservice boundaries, standalone Deployments, baseline persistent volume requirements, and automated backup loops.

    overlays/stateful/: Utilises strategic merge deletion patches to safely migrate your persistent database engine from a generic Deployment into a robust, ordered StatefulSet bound to a headless cluster service interface.

    overlays/stateful-hpa/: Introduces metrics-driven automation rules by applying native HorizontalPodAutoscaler configurations to scale computing pods relative to live demand.

    overlays/stateful-hpa-probes/: Injects self-healing diagnostic loops by instrumenting standalone HTTP /livez and /readyz validation points.

    overlays/production/: The final infrastructure environment layer incorporating path-based ingress tracking, base64 binary configuration maps for profile assets, and cluster network isolation firewalls.

2. Zero-Trust Network Topologies (NetworkPolicies)

Enforces strict ingress and egress segregation controls across your cluster's software-defined network (SDN):

    The database tier rejects all native routing; traffic parameters are whitelisted to accept entry only from verified application microservice layers over port 3306.

    Web presentation instances are restricted from lateral internal scanning, strictly locking egress targets to specific database paths and core cluster DNS lookups (kube-dns:53).

3. Native Package Automation (/charts)

The entire multi-frontend stack layout is completely abstracted into a fully parameterized Helm v3 Chart published to an OCI container registry (ghcr.io), allowing clean, single-command bootstrapping.

4. Custom API Extensions (/manifests/crds)

Extends the native Kubernetes API machinery by declaring a custom CustomResourceDefinition (CRD) tracking specification to handle custom resource models directly inside the cluster state engine (etcd).
