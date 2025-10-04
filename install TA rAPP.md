# Joe's TA rAPP Installation log
## Environment
- macOS (ARM), Ubuntu VM(ARM64) 22.04
- K8s : MicroK8s v1.29.x

## install MicroK8s
- what is K8s
  - K8s is the abbreviation for Kubernetes.
  - It is a container orchestration platform used to automate the management, deployment, scaling, and operation of containerized applications.
- what can K8s do?
  - Deployment: You can easily package an application into a Docker container and deploy it to multiple machines through K8s.
  - Scaling: Pods are automatically added when there are many users, and automatically shut down when there are fewer users, saving resources.
  - High Availability: When a node fails, K8s will automatically move the container to other healthy nodes.
  - Self-healing: If the container crashes, K8s will automatically restart it.
- Core Concepts of K8s
  - Pod: The smallest unit, typically a container or a group of closely related containers.
  - Node: The machine that runs a pod (can be a VM or a physical machine).
  - Cluster: The entire Kubernetes environment, consisting of a group of nodes.
  - Deployment: Describes how an application should run (number of replicas, container image used, etc.).
  - Service: The abstraction layer that provides external services and is responsible for traffic distribution.

- What is MicroK8s
  - It is a lightweight version of Kubernetes launched by Canonical (the company behind Ubuntu)
 
- install steps
```
# Install MicroK8s (lightweight Kubernetes) via Snap, pin to v1.29 stable; 
# --classic grants the confinement needed by MicroK8s daemons
sudo snap install microk8s --channel=1.29/stable --classic

# Add your current Linux user into the "microk8s" group
# so you can run microk8s/kubectl/helm without sudo
sudo usermod -aG microk8s $USER

# Reload your shell with the updated group membership immediately
# (alternative: log out and log back in)
newgrp microk8s

# Wait until the MicroK8s control plane and core add-ons are fully ready
# This blocks until Ready, which makes later commands (kubectl/helm) reliable
microk8s status --wait-ready
```
<img width="875" height="540" alt="image" src="https://github.com/user-attachments/assets/1593f1a0-f8e6-4b59-93c1-c24f18985a99" />

<img width="705" height="93" alt="image" src="https://github.com/user-attachments/assets/73cbe509-e4c6-4824-a200-215caa243e5f" />

- microk8s.kubectl get pods -A # **View the status of all pods**
- microk8s helm3 list -A # **View installed Helm releases**

## Helm

Helm is **the package manager for Kubernetes**. Like `apt/yum` on Linux or `brew` on macOS, Helm lets you **install, upgrade, roll back, and remove** apps on a Kubernetes cluster.

---
- Core Concepts
- **Chart** — a Helm package (folder or `.tgz`) containing:
  - `Chart.yaml` — metadata (name, version, dependencies)
  - `values.yaml` — default configuration (can be overridden)
  - `templates/` — Kubernetes YAML templates (Deployment, Service, etc.)
- **Repository** — a collection of charts (e.g., Bitnami).
- **Release** — a **deployed instance** of a chart in your cluster.
- **Values** — parameters you pass at install/upgrade time to customize a chart.

---

## build and install
<img width="528" height="81" alt="image" src="https://github.com/user-attachments/assets/8be5df30-71fc-49a4-a4c5-ce01bcbb341c" />

- **I use Docker instead of nerdctl to build and push images**
- 1.install Docker
  - <img width="898" height="20" alt="image" src="https://github.com/user-attachments/assets/f3151a8c-a233-4fda-aad8-edf0a01f6126" />
- 2.build
  - <img width="875" height="39" alt="image" src="https://github.com/user-attachments/assets/54444087-ba62-4765-b969-4bfe9f9f6f96" />
- 3.push
  - <img width="875" height="38" alt="image" src="https://github.com/user-attachments/assets/e03d603b-cdd8-4a65-a57d-7c25b120df49" />
- 4.install
  - <img width="876" height="254" alt="image" src="https://github.com/user-attachments/assets/88eb1664-4210-48da-a478-f6e31ac6bdee" />

```
microk8s.kubectl get svc -n rapp #Use kubectl (MicroK8s version) to list all services in the namespace rapp in Kubernetes
microk8s.kubectl get pods -n rapp -w #List all Pods in the rapp namespace
```

- <img width="953" height="75" alt="image" src="https://github.com/user-attachments/assets/e688bc7a-598b-42b6-a60f-f45b63b55735" />
- **Stuck here for a while**
  - This means both Pods are still being created and are not yet ready for use:
  - READY 0/1: The container is not ready yet (it should be ready only when it reaches 1/1).
  - STATUS ContainerCreating: Kubernetes is pulling images, mounting volumes, configuring networking, etc.
 
```
# Check the Pod's events (displayed starting after "Events:") to quickly identify the cause of the stall.
microk8s.kubectl -n rapp describe pod ta-rapp-nonrtric-rapp-test-automation-5bb8bddb4d-gqgzf | awk '/Events:/,0'

# Create the required directories for the hostPath (missing them will result in a FailedMount)
sudo mkdir -p /tmp/viavi/o1/pm_reports

# Temporarily grant read/write permissions to the container (for troubleshooting; you can tighten these permissions later based on actual needs)
sudo chmod -R 777 /tmp/viavi

# Re-roll the deployment (rebuild the new Pod and retry the mount)
microk8s.kubectl -n rapp rollout restart deploy/ta-rapp-nonrtric-rapp-test-automation

# Continue to monitor the Pod's status until you see READY 1/1, STATUS Running (press Ctrl-C to end)
microk8s.kubectl get pods -n rapp -w

```

## Upload Test Specification to TA rApp
```
cd nonrtric-rapp-test-automation/Test-Automation-rApp/src/config/
curl -X POST "http://192.168.1.115:30082/upload_test_spec" \
  -H "Content-Type: application/json" \
  --data-binary @test_spec.json
```
<img width="871" height="234" alt="image" src="https://github.com/user-attachments/assets/11dd200a-588f-4277-bfef-1575e0704201" />
