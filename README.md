# Self-Managed Kubernetes — kubeadm & Cluster Operations

**Kubernetes · Linux · kubeadm · containerd · Calico · Networking · Troubleshooting**

Built and operated a **two-node, self-managed Kubernetes cluster** on Ubuntu virtual machines using kubeadm, containerd and Calico — working directly with the underlying infrastructure rather than relying on a managed Kubernetes service.

The project covers cluster bootstrapping, node networking, application deployment, Kubernetes Services, workload health checks, scaling, self-healing, rolling updates, rollback and troubleshooting.

I built this environment independently after completing an AWS EKS project because I wanted to understand what happens underneath managed Kubernetes: how nodes join a cluster, how pod networking works, how workloads are scheduled and how to recover when components fail.

**Verified outcome:** A working Kubernetes v1.35.9 control plane and worker node, with **four healthy NGINX replicas** accessible from the Windows host through a NodePort Service.

The repository includes Kubernetes manifests, an architecture diagram and **19 screenshots** documenting the cluster configuration, application deployment, operational tests and troubleshooting.

**[Architecture](#architecture)** · **[Implementation Evidence](#implementation-evidence)** · **[Deployment Guide](#deploy-the-application)** · **[Troubleshooting](#troubleshooting-and-lessons-learned)**

---

## Project Walkthrough

A technical walkthrough covering the self-managed Kubernetes architecture, cluster bootstrap with kubeadm, container networking, NGINX deployment, scaling, self-healing, rolling updates and troubleshooting.

[![Watch the Self-Managed Kubernetes Project Walkthrough](https://img.youtube.com/vi/a9hATioCGcs/hqdefault.jpg)](https://youtu.be/a9hATioCGcs)

**[▶ Watch the Kubernetes kubeadm Project Walkthrough (2:00)](https://youtu.be/a9hATioCGcs)**

---

## Infrastructure Overview

| Component | Implementation |
| --- | --- |
| **Infrastructure** | 2 Ubuntu 26.04.1 LTS VMs on Oracle VirtualBox |
| **Kubernetes** | v1.35.9, bootstrapped with kubeadm |
| **Control Plane** | `k8s-control` |
| **Worker Node** | `k8s-worker` |
| **Container Runtime** | containerd 2.2.2 |
| **Container Network Interface** | Calico v3.31.0 |
| **VM Networking** | NAT and host-only networking |
| **Application** | NGINX |
| **Application Replicas** | 4 on the worker node |
| **Kubernetes Services** | ClusterIP and NodePort |
| **Health Management** | Readiness/liveness probes |
| **Resources** | CPU/memory requests and limits |
| **Operations Tested** | Scaling, pod self-healing, rolling updates, rollback |
| **Configuration** | Declarative Kubernetes YAML manifests |
| **Deployment Evidence** | 19 screenshots and architecture diagram |

This is a local Kubernetes infrastructure lab, not a commercial production deployment.

---

## Architecture

![Self-Managed Kubernetes Architecture](screenshots/59-kubeadm-kubernetes-architecture.png)

### Cluster Nodes

The lab consists of two Ubuntu virtual machines running on Oracle VirtualBox.

| Node | Responsibility | Host-only IP |
| --- | --- | --- |
| `k8s-control` | Kubernetes control plane | `192.168.56.10` |
| `k8s-worker` | Worker node running NGINX workloads | `192.168.56.11` |

### Control Plane

The control-plane node runs the Kubernetes components responsible for managing cluster state and coordinating workloads.

It provides the Kubernetes API through which cluster administration commands are issued.

I initialised this node using `kubeadm` before joining the worker node.

### Worker Node

The worker node runs the application workloads.

It uses containerd as the container runtime and Calico for pod networking.

The NGINX Deployment was configured to run four replicas, which were scheduled on the worker node.

### How Traffic Reaches the Application

**Access from the Windows host**

```text
Windows Host
     |
     | HTTP :32396
     v
Worker Node — 192.168.56.11
     |
     v
NodePort Service — nginx-nodeport
     |
     v
NGINX Pods
```

**Internal Kubernetes access**

```text
In-cluster Client
     |
     v
ClusterIP Service — nginx-service :80
     |
     v
NGINX Pods
```

The ClusterIP and NodePort Services provide separate access paths to the same NGINX workload.

Traffic does not need to pass through the ClusterIP Service before reaching the NodePort Service.

### Cluster Administration

`kubectl` communicates with the Kubernetes API server on the control-plane node.

The Kubernetes control plane manages the desired state of workloads, while the worker node runs the containers.

**Note:** The IP addresses and NodePort shown in this README belong to my local VirtualBox environment. Other environments may use different addresses or ports.

---

## VM Networking Configuration

Both virtual machines use a networking configuration designed to support a local Kubernetes environment.

### NAT Networking

NAT provides outbound connectivity for:

- Downloading packages
- Installing Kubernetes components
- Retrieving container images
- Accessing external repositories

### Host-Only Networking

Host-only networking enables communication between the Windows host and the Kubernetes VMs.

It provides the network path used for administration and accessing the NGINX application through NodePort.

### Static IP Addresses

I configured predictable IP addresses for the two nodes:

- Control plane: `192.168.56.10`
- Worker: `192.168.56.11`

This simplified SSH access, node connectivity and application testing.

### Networking Evidence

| Configuration | Evidence |
| --- | --- |
| Control-plane NAT adapter | [View screenshot](screenshots/03-control-plane-nat.png) |
| Control-plane host-only adapter | [View screenshot](screenshots/04-control-plane-hostonly.png) |
| Static control-plane IP | [View screenshot](screenshots/06-control-plane-static-ip.png) |

The worker node was configured with `192.168.56.11` on the host-only network.

---

## Engineering Implementation

### 1. Infrastructure Preparation

Created the underlying virtual infrastructure before installing Kubernetes.

- Provisioned Ubuntu VMs using Oracle VirtualBox.
- Configured NAT and host-only networking.
- Assigned static IP addresses.
- Installed and configured containerd.
- Installed Kubernetes tooling.
- Initialised the control plane using `kubeadm`.
- Joined the worker node to the cluster.

This provided experience working with the infrastructure required to run Kubernetes, rather than starting with a preconfigured cluster.

### 2. Kubernetes Bootstrap and Networking

After preparing the VMs, I configured the Kubernetes environment.

- Bootstrapped the control plane using kubeadm.
- Configured containerd as the container runtime.
- Installed Calico as the Container Network Interface.
- Verified Kubernetes system components.
- Joined the worker node.
- Confirmed both nodes reached the `Ready` state.

This stage helped me understand the relationship between the container runtime, CNI, Kubernetes API and node readiness.

### 3. Application Deployment

Deployed NGINX using declarative Kubernetes YAML manifests.

Configured:

- Kubernetes Deployment
- ClusterIP Service
- NodePort Service
- Readiness and liveness probes
- CPU and memory resource requests
- CPU and memory resource limits

The workload was successfully scheduled on the Kubernetes worker node.

### 4. Service Networking and Connectivity

Exposed the NGINX application through two Kubernetes Services.

**ClusterIP:** Provides internal service connectivity within Kubernetes.

**NodePort:** Makes the application accessible through the node's IP address and allocated port.

Verified application access from the Windows host and tested connectivity after application updates.

### 5. Kubernetes Operations

Performed operational exercises against the cluster, including:

- Scaling the Deployment to four replicas
- Deleting a pod and observing automatic replacement
- Performing a rolling application update
- Checking rollout status
- Restoring a previous Deployment revision
- Verifying application connectivity after the rollback

These tests demonstrate Kubernetes Deployment management and basic workload recovery.

### 6. Infrastructure Troubleshooting

Worked through issues affecting the underlying Kubernetes environment, including:

- VirtualBox CPU performance problems
- Kubernetes bootstrap failures
- Cluster administrative access
- RBAC-related investigation
- Kubernetes component recovery and readiness checks

These problems required investigating the environment underneath the application manifests.

---

## Implementation Evidence

The following **19 screenshots** document the infrastructure configuration, Kubernetes installation, application deployment, operational tests, troubleshooting and final architecture.

### 1. Kubernetes Cluster Running

Both Kubernetes nodes successfully joined the cluster and reached the `Ready` state.

![Two Kubernetes Nodes Ready](screenshots/49-two-node-kubernetes-cluster-ready.png)

**Supporting evidence**

- [Containerd configured](screenshots/16-control-plane-containerd.png)
- [Kubernetes tools installed](screenshots/22-kubernetes-tools-installed.png)
- [Calico CNI installed](screenshots/42-calico-cni-installation.png)
- [Kubernetes system pods running](screenshots/43-kubernetes-system-pods-running.png)

### 2. Application Deployment and Networking

NGINX was deployed on the worker node and exposed using Kubernetes Services.

The application was accessible from the Windows host through NodePort.

![NGINX Accessible Through NodePort](screenshots/53-nginx-nodeport-browser-access.png)

**Supporting evidence**

- [NGINX pods on worker node](screenshots/51-nginx-workloads-on-worker.png)
- [ClusterIP connectivity](screenshots/52-nginx-clusterip-service-connectivity.png)

### 3. Kubernetes Operations

The cluster was used to test workload management, recovery and deployment changes.

| Test | Evidence | Result |
| --- | --- | --- |
| **Pod self-healing** | [Pod replacement](screenshots/54-kubernetes-self-healing-replacement-pod.png) | Replacement pod created after deletion |
| **Scaling** | [Four replicas](screenshots/55-nginx-horizontal-scaling-four-replicas.png) | Four NGINX replicas running |
| **Rolling update** | [Successful rollout](screenshots/56-nginx-rolling-update-success.png) | Deployment update completed |
| **Post-update validation** | [Connectivity check](screenshots/57-nginx-post-update-connectivity.png) | Application remained accessible |
| **Rollback** | [Rollback validation](screenshots/58-nginx-rollback-connectivity.png) | Previous revision restored and tested |

These exercises provided direct experience with Kubernetes Deployment behaviour and workload reconciliation.

### 4. Troubleshooting Evidence

**Kubernetes administrative access and RBAC**

[View Kubernetes RBAC recovery screenshot](screenshots/28-kubernetes-rbac-recovery.png)

**VirtualBox performance and hardware acceleration**

[View native VT-x acceleration confirmation](screenshots/37-native-vtx-acceleration-confirmed.png)

### 5. Architecture Evidence

The architecture diagram documents the two-node cluster, VM networking, Kubernetes Services and NGINX deployment.

[View full architecture diagram](screenshots/59-kubeadm-kubernetes-architecture.png)

---

## Design Decisions

| Decision | Reason |
| --- | --- |
| **Use kubeadm** | Understand cluster bootstrap and administration rather than relying on a managed service |
| **Separate control plane and worker node** | Understand node responsibilities and workload scheduling |
| **Use Ubuntu VMs** | Work directly with Linux infrastructure supporting Kubernetes |
| **Use NAT and host-only networking** | Provide outbound connectivity and local access |
| **Use static node IPs** | Simplify administration, SSH and networking tests |
| **Use containerd** | Run Kubernetes workloads using a container runtime |
| **Use Calico** | Provide Kubernetes pod networking |
| **Use declarative manifests** | Maintain application configuration in version control |
| **Configure ClusterIP and NodePort** | Demonstrate internal networking and access from the host |
| **Use readiness/liveness probes** | Understand application availability and container health checks |
| **Use CPU/memory requests and limits** | Practise workload resource configuration |
| **Test scaling and rollbacks** | Understand workload management and deployment recovery |

### Why kubeadm instead of another managed Kubernetes service?

I had already built a Kubernetes platform using Amazon EKS, which manages the Kubernetes control plane.

For this project, I wanted to work more directly with the components that a managed service normally handles.

Using kubeadm required me to configure the control plane and worker node, install the container runtime, configure pod networking and troubleshoot bootstrap and access issues.

This made the lab a useful complement to EKS rather than another version of the same project.

---

## Repository Structure

```text
kubeadm-kubernetes-lab/
├── README.md
├── manifests/
│   ├── deployment.yaml
│   ├── service-clusterip.yaml
│   └── service-nodeport.yaml
└── screenshots/
    ├── 03-control-plane-nat.png
    ├── 04-control-plane-hostonly.png
    ├── 06-control-plane-static-ip.png
    ├── 16-control-plane-containerd.png
    ├── 22-kubernetes-tools-installed.png
    ├── 28-kubernetes-rbac-recovery.png
    ├── 37-native-vtx-acceleration-confirmed.png
    ├── 42-calico-cni-installation.png
    ├── 43-kubernetes-system-pods-running.png
    ├── 49-two-node-kubernetes-cluster-ready.png
    ├── 51-nginx-workloads-on-worker.png
    ├── 52-nginx-clusterip-service-connectivity.png
    ├── 53-nginx-nodeport-browser-access.png
    ├── 54-kubernetes-self-healing-replacement-pod.png
    ├── 55-nginx-horizontal-scaling-four-replicas.png
    ├── 56-nginx-rolling-update-success.png
    ├── 57-nginx-post-update-connectivity.png
    ├── 58-nginx-rollback-connectivity.png
    └── 59-kubeadm-kubernetes-architecture.png
```

The repository contains the Kubernetes application manifests and implementation evidence.

Cluster bootstrap was performed separately and is documented through the project description and screenshots. The repository does not contain a fully automated cluster provisioning process.

---

## Deploy the Application

These instructions deploy the NGINX application and Services to an **existing Kubernetes cluster**.

They do not create the VirtualBox VMs or bootstrap a new Kubernetes cluster.

### Prerequisites

A functioning Kubernetes environment with:

- `kubectl` installed and configured
- Kubernetes nodes in the `Ready` state
- Working container runtime
- Working CNI networking
- Permission to create Deployments and Services

### 1. Clone the Repository

```bash
git clone https://github.com/ismaaeelahmed11/kubeadm-kubernetes-lab.git

cd kubeadm-kubernetes-lab
```

### 2. Check Cluster Connectivity

```bash
kubectl cluster-info

kubectl get nodes -o wide

kubectl get pods -A
```

Confirm the API server is accessible and the nodes are reporting `Ready`.

### 3. Validate the Manifests

```bash
kubectl apply --dry-run=client -f manifests/
```

Review the YAML configuration before deploying.

### 4. Deploy NGINX

```bash
kubectl apply -f manifests/
```

### 5. Verify the Application

```bash
kubectl get deployments

kubectl get pods -o wide

kubectl get services
```

Check the rollout:

```bash
kubectl rollout status deployment/nginx-demo
```

### 6. Access NGINX

From the Windows host connected to the VirtualBox host-only network:

```text
http://192.168.56.11:32396
```

The IP address and NodePort are specific to my lab.

For another environment, inspect the current Service configuration:

```bash
kubectl get services
```

---

## Operational Tests

These operations were performed against the running Kubernetes lab.

### 1. Horizontal Scaling

Scaled the NGINX Deployment to four replicas:

```bash
kubectl scale deployment nginx-demo --replicas=4
```

Verified the pods:

```bash
kubectl get deployments

kubectl get pods -o wide
```

**Result:** Four NGINX replicas were running on the Kubernetes worker node.

This demonstrated manual horizontal scaling through the Kubernetes Deployment controller.

A Horizontal Pod Autoscaler was not configured.

### 2. Pod Self-Healing

Deleted an NGINX pod to observe how the Deployment controller responded.

Kubernetes uses a ReplicaSet to maintain the desired number of replicas.

Monitored the application:

```bash
kubectl get pods -l app=nginx-demo -w
```

**Result:** Kubernetes created a replacement pod after deletion, restoring the desired replica count.

This tested recovery from an individual pod deletion, not failure of the worker node.

### 3. Rolling Updates

Performed a rolling update by changing the NGINX container image version.

Monitored progress:

```bash
kubectl rollout status deployment/nginx-demo
```

Inspected the Deployment revision history:

```bash
kubectl rollout history deployment/nginx-demo
```

**Result:** The Deployment completed its rollout, and application connectivity was verified afterwards.

### 4. Deployment Rollback

Restored a previous Deployment revision:

```bash
kubectl rollout undo deployment/nginx-demo
```

Checked the rollout:

```bash
kubectl rollout status deployment/nginx-demo
```

**Result:** The rollback completed and NGINX connectivity was successfully tested.

> These exercises used Kubernetes commands against the running lab. Where imperative changes are intended to persist, they should also be reflected in the declarative YAML configuration stored in Git.

---

## Troubleshooting and Lessons Learned

The project involved troubleshooting the virtualisation environment, Kubernetes bootstrap and cluster administration.

These issues reinforced how Kubernetes depends on the underlying operating system, networking and compute infrastructure.

### 1. VirtualBox Performance and Kubernetes Bootstrap

**Problem**

During Kubernetes setup, the control-plane VM experienced CPU lockups and Kubernetes components stopped progressing normally.

**Investigation**

- Checked VM resource allocation and performance.
- Investigated host virtualisation settings.
- Examined Kubernetes component behaviour during bootstrap.
- Checked whether native hardware acceleration was available.

**Resolution**

Recovered the VM environment with native hardware virtualisation available and restored normal Kubernetes operation.

**Evidence:** [Native VT-x acceleration confirmation](screenshots/37-native-vtx-acceleration-confirmed.png)

**What I took from it**

The Kubernetes control plane depends on the stability of the infrastructure underneath it.

CPU virtualisation, VM performance and host configuration can prevent Kubernetes components from operating correctly even when the Kubernetes configuration itself appears valid.

### 2. Kubernetes Bootstrap and RBAC

**Problem**

Kubernetes API access and administrative permissions required investigation during the initial setup and recovery process.

**Investigation**

- Examined Kubernetes administrative access.
- Investigated authentication and RBAC configuration.
- Worked through cluster recovery.
- Verified Kubernetes component status after recovering access.

**Resolution**

Recovered administrative access and continued the Kubernetes setup.

**Evidence:** [Kubernetes RBAC recovery](screenshots/28-kubernetes-rbac-recovery.png)

**What I took from it**

Cluster administration depends on both authentication and authorisation.

Being able to reach the Kubernetes API server does not necessarily mean a user has permission to perform the requested operation.

### 3. Understanding Workload Scheduling

**Observation**

All four NGINX replicas were scheduled on the same worker node.

**What I took from it**

Multiple replicas provide recovery from individual pod failures, but they do not provide redundancy against a complete worker-node failure when every pod runs on that node.

A more resilient design would require additional worker nodes, appropriate scheduling configuration and actual failure testing.

### 4. Kubernetes Service Networking

**Observation**

NGINX was exposed using separate ClusterIP and NodePort Services.

Both Services routed requests to the application workload.

**What I took from it**

ClusterIP provides internal Kubernetes service access.

NodePort exposes a Service through a node's IP address and allocated port, subject to the surrounding network configuration.

Understanding the relationship between Services, pod endpoints and node networking was important when validating application access.

---

## Scope and Limitations

This project is a **self-managed Kubernetes homelab**, built to develop cluster administration and troubleshooting skills.

It is not presented as a high-availability or commercially operated production environment.

The main limitations are:

- **Single control plane:** The Kubernetes API is not highly available.
- **Single worker node:** All four NGINX replicas depend on the same worker.
- **No node-level failover:** Worker-node failure can make the application unavailable.
- **Local networking:** NodePort access is through the VirtualBox host-only environment, not the public internet.
- **Manual infrastructure setup:** VirtualBox provisioning and kubeadm bootstrap are not automated in this repository.
- **Manual scaling:** Four replicas were configured through a scaling exercise, not an HPA.
- **Limited operational testing:** Pod replacement, rolling updates and rollback were tested, but multi-node failover, disaster recovery and long-term production reliability were not.

These limitations are intentional within the project's scope.

The main objective was understanding and operating Kubernetes infrastructure directly.

---

## What This Project Demonstrates

This lab gave me practical experience working with Kubernetes beyond managed cloud platforms.

**Infrastructure and administration**

- Preparing Ubuntu virtual machines
- Configuring node networking
- Installing containerd
- Bootstrapping Kubernetes with kubeadm
- Joining and verifying a worker node
- Configuring Calico pod networking

**Kubernetes application operations**

- Deploying applications through declarative YAML
- Configuring ClusterIP and NodePort Services
- Managing readiness and liveness probes
- Applying CPU and memory resource settings
- Scaling workloads manually
- Testing pod self-healing
- Performing rolling updates and rollback

**Troubleshooting**

- Investigating VM CPU lockups
- Working through Kubernetes bootstrap issues
- Recovering cluster administrative access
- Investigating RBAC configuration
- Verifying Kubernetes system components and node readiness

### Final Outcome

Successfully built and operated a **two-node Kubernetes v1.35.9 cluster** on Ubuntu virtual machines, with containerd and Calico configured and four NGINX replicas accessible through NodePort.

The most valuable part was understanding the underlying Kubernetes environment and working through the failures encountered during setup.

This project complements my AWS EKS platform by demonstrating direct experience with the cluster components and administration tasks that a managed Kubernetes service normally handles.

---

## Related Engineering Projects

### [AWS EKS — End-to-End Kubernetes, GitOps & Monitoring](https://github.com/ismaaeelahmed11/eks-project)

A Terraform-provisioned EKS platform with NGINX Ingress, cert-manager, ExternalDNS, Argo CD, GitHub Actions and Prometheus/Grafana monitoring.

**[View EKS Project →](https://github.com/ismaaeelahmed11/eks-project)** · **[Watch Walkthrough →](https://youtu.be/RDmnaTi2auE)**

### [AWS ECS Fargate — End-to-End Infrastructure & CI/CD](https://github.com/ismaaeelahmed11/devops-learning/tree/main/ecs-project)

A containerised Go application deployed to AWS ECS Fargate using Terraform, Docker, GitHub Actions, HTTPS and CloudWatch.

**[View ECS Project →](https://github.com/ismaaeelahmed11/devops-learning/tree/main/ecs-project)** · **[Watch Walkthrough →](https://youtu.be/qpIZlJkWmos)**

---

## Connect

**Ismaaeel Ahmed — DevOps & Cloud Engineering**

[GitHub Portfolio](https://github.com/ismaaeelahmed11) · [LinkedIn](https://www.linkedin.com/in/ismaaeelahmed) · [Email](mailto:ismaaeelahmed11@gmail.com)
