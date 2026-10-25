# Self-Managed Kubernetes — kubeadm, Linux & Cluster Operations

**Kubernetes · Linux (Ubuntu) · kubeadm · containerd · Calico · VirtualBox · Networking · YAML · Troubleshooting**

> **Completed hands-on Kubernetes infrastructure lab**  
> Built and operated a **two-node Kubernetes cluster** from Ubuntu virtual machines instead of using a managed Kubernetes control plane. Demonstrated working node connectivity, application access, **four NGINX replicas**, pod self-healing, rolling updates and rollback. The source manifests, walkthrough and **19 implementation screenshots** are preserved in this repository.

## Project Overview

After building a Kubernetes platform on **Amazon EKS**, I wanted to understand and troubleshoot the infrastructure underneath it. Rather than starting with a ready-made cluster, I prepared **two Ubuntu VMs in Oracle VirtualBox**, configured their networking and container runtime, bootstrapped Kubernetes with **kubeadm**, installed **Calico** for pod networking and joined a worker node.

I then deployed **NGINX** from declarative Kubernetes manifests, exposed it internally via **ClusterIP** and to my Windows host via **NodePort**, and tested the Kubernetes Deployment controller by **scaling, deleting a pod, updating a workload and rolling back a revision**.

This project demonstrates **Linux-based infrastructure administration, Kubernetes cluster setup, networking, workload operations and troubleshooting**—skills relevant to junior **DevOps, Platform, Cloud and Infrastructure Engineering** roles. It complements my cloud-managed [AWS EKS project](https://github.com/ismaaeelahmed11/eks-project) rather than repeating it.

### Verified Project Outcomes

| Area | Demonstrated outcome |
| --- | --- |
| **Self-managed Kubernetes** | A **control plane and worker node** bootstrapped with kubeadm, both observed `Ready` |
| **Linux infrastructure** | Two Ubuntu VMs with NAT and host-only adapters and predictable node addresses |
| **Container runtime / CNI** | containerd configured; Calico installed for Kubernetes pod networking |
| **Application deployment** | Declarative NGINX Deployment with **4 replicas**, probes and resource controls |
| **Service networking** | Two Services selecting the same Pods: **ClusterIP** for in-cluster access and **NodePort** for host access |
| **Live access** | NGINX accessible from Windows at `http://192.168.56.11:32396` during the lab |
| **Workload recovery** | Deleted a Pod and observed the Deployment controller create a replacement |
| **Release operations** | Performed a rolling update, checked application connectivity and rolled back |
| **Troubleshooting** | Investigated virtualization/CPU lockups, Kubernetes bootstrap, administrative access and RBAC |
| **Evidence** | **19 screenshots**, architecture diagram, Kubernetes YAML and a **2-minute video walkthrough** |

> **Scope:** This is a **self-managed Kubernetes homelab using real Linux VMs**, not a commercial production cluster. It proves practical setup and workload operations; it does **not** claim node-level high availability, an automated kubeadm upgrade, disaster recovery or long-term production operation.

---

## Watch the Project Walkthrough

[![Watch the self-managed Kubernetes kubeadm walkthrough](https://img.youtube.com/vi/a9hATioCGcs/hqdefault.jpg)](https://youtu.be/a9hATioCGcs)

**[▶ Watch the 2:00 technical walkthrough](https://youtu.be/a9hATioCGcs)** — VirtualBox networking, kubeadm bootstrap, Calico, NGINX, service access, scaling, self-healing, rollout and troubleshooting.

## Architecture

![Two-node self-managed Kubernetes architecture](screenshots/59-kubeadm-kubernetes-architecture.png)

### Cluster layout

| Node | Role | Host-only address |
| --- | --- | --- |
| `k8s-control` | Kubernetes control plane / API administration | `192.168.56.10` |
| `k8s-worker` | Worker node hosting the NGINX Pods | `192.168.56.11` |

Both machines run on **Oracle VirtualBox**. **NAT** provides outbound connectivity for package and image downloads, while a **host-only network** connects the Windows host to the Ubuntu VMs and supports administration and NodePort testing.

### Application request paths

```text
Windows host browser
       |
       | http://192.168.56.11:32396
       v
k8s-worker (host-only address)
       |
       v
nginx-nodeport Service :32396 -> targetPort :80
       |
       v
NGINX Pods (selected by app=nginx-demo)
```

```text
In-cluster client
       |
       v
nginx-service (ClusterIP :80 -> targetPort :80)
       |
       v
The same NGINX Pods (app=nginx-demo)
```

The **ClusterIP and NodePort Services are separate entry points**, each targeting the same Pod label selector. NodePort traffic does **not** need to flow through the separate ClusterIP Service. The **Kubernetes API server** is the administration/control interface; user HTTP requests reach the workload through the Kubernetes Service networking path.

**Lab-specific details:** The addresses `192.168.56.10`, `192.168.56.11` and NodePort `32396` belong to this VirtualBox environment. The application was **not exposed to the public internet**.

---

## Proof of Implementation — Start Here

The screenshots below establish the important outcome quickly. The full **19-image evidence inventory** is organised by engineering area in [Implementation Evidence](#implementation-evidence).

| **1. Two Kubernetes nodes Ready** | **2. Application reached from Windows** |
| --- | --- |
| ![Kubernetes control plane and worker nodes Ready](screenshots/49-two-node-kubernetes-cluster-ready.png) | ![NGINX accessed through NodePort in Windows browser](screenshots/53-nginx-nodeport-browser-access.png) |
| kubeadm cluster running with control-plane and worker roles | Host-to-VM networking and NodePort request path working |

| **3. Pod replacement / self-healing** | **4. Deployment rollback verified** |
| --- | --- |
| ![NGINX pod self-healing and replacement](screenshots/54-kubernetes-self-healing-replacement-pod.png) | ![NGINX application working following rollback](screenshots/58-nginx-rollback-connectivity.png) |
| Replacement Pod observed after deleting a replica | Previous Deployment revision restored and connectivity checked |

**Additional operational proof:** [Four NGINX replicas](screenshots/55-nginx-horizontal-scaling-four-replicas.png) · [Rolling-update result](screenshots/56-nginx-rolling-update-success.png) · [Connectivity after update](screenshots/57-nginx-post-update-connectivity.png) · [Kubernetes system Pods](screenshots/43-kubernetes-system-pods-running.png)

> These are **point-in-time screenshots** of observed results. They are not a claim that the lab is continuously running, that a worker-node failure was survived, or that a Kubernetes version upgrade was completed.

---

## Engineering Skills — Linked to Code and Evidence

| Skill | What I implemented or tested | Evidence |
| --- | --- | --- |
| **Linux / infrastructure** | Prepared Ubuntu VMs, network interfaces, static addressing and runtime prerequisites | [Static IP configuration](screenshots/06-control-plane-static-ip.png), [VM networking](screenshots/04-control-plane-hostonly.png) |
| **Kubernetes cluster administration** | kubeadm bootstrap, worker-node join and readiness verification | [Two nodes Ready](screenshots/49-two-node-kubernetes-cluster-ready.png), [system Pods](screenshots/43-kubernetes-system-pods-running.png) |
| **Container runtime** | Configured containerd to support Kubernetes workloads | [containerd evidence](screenshots/16-control-plane-containerd.png) |
| **Kubernetes networking** | Installed Calico; configured ClusterIP and NodePort access | [Calico](screenshots/42-calico-cni-installation.png), [ClusterIP connectivity](screenshots/52-nginx-clusterip-service-connectivity.png) |
| **Declarative configuration** | Authored a Deployment with replica count, rollout strategy, probes and resources | [`manifests/deployment.yaml`](manifests/deployment.yaml) |
| **Service discovery / exposure** | Created separate Services with the same Pod selector | [ClusterIP YAML](manifests/service-clusterip.yaml), [NodePort YAML](manifests/service-nodeport.yaml) |
| **Kubernetes operations** | Tested scaling, controller reconciliation, rolling updates and rollback | [Scale](screenshots/55-nginx-horizontal-scaling-four-replicas.png), [self-healing](screenshots/54-kubernetes-self-healing-replacement-pod.png), [rollout](screenshots/56-nginx-rolling-update-success.png) |
| **Troubleshooting** | Investigated virtualization performance, bootstrap, permissions and API access | [VT-x evidence](screenshots/37-native-vtx-acceleration-confirmed.png), [RBAC investigation](screenshots/28-kubernetes-rbac-recovery.png) |
| **Documentation** | Recorded architecture, actual manifests, screenshots and walkthrough | [Architecture](screenshots/59-kubeadm-kubernetes-architecture.png), [walkthrough](https://youtu.be/a9hATioCGcs) |

---

## Technical Environment

| Component | Lab configuration |
| --- | --- |
| Host | Windows machine running Oracle VirtualBox |
| Guest OS | **Ubuntu 26.04.1 LTS**, on two VMs (as documented during the build) |
| Kubernetes | **v1.35.9**, installed using kubeadm (recorded project version) |
| Control plane | `k8s-control` |
| Worker | `k8s-worker` |
| Container runtime | **containerd 2.2.2** (recorded project version) |
| CNI | **Calico v3.31.0** (recorded project version) |
| VM networking | NAT plus host-only interfaces; static host-only addresses |
| Workload | NGINX `nginx:stable` |
| Desired workload size | **4 replicas** |
| Internal Service | `nginx-service`, ClusterIP port `80` |
| Host-access Service | `nginx-nodeport`, NodePort `32396` → target port `80` |
| Deployment strategy | `RollingUpdate`, `maxSurge: 1`, `maxUnavailable: 0` |
| Readiness / liveness | HTTP `GET /` on container port `80` |
| CPU request / limit | `100m` / `500m` per container |
| Memory request / limit | `64Mi` / `128Mi` per container |

These workload settings are taken from the committed [Deployment](manifests/deployment.yaml) and [Services](manifests/service-clusterip.yaml). The Kubernetes, OS, containerd and Calico version numbers are **documented build versions**; the repository does not contain an automated image or version audit for reproducing them later.

---

## Building the Cluster — From VMs to `Ready`

### 1. Prepare the Ubuntu infrastructure

I created two Ubuntu VMs on VirtualBox and configured both **NAT** and **host-only** networking. I used fixed addresses on the host-only adapter so that node communication and host-based tests did not depend on changing IP assignments.

- `k8s-control` — `192.168.56.10`
- `k8s-worker` — `192.168.56.11`

I checked the VM networking, Linux prerequisites and underlying host virtualization when the initial environment encountered performance issues.

**Evidence:** [NAT adapter](screenshots/03-control-plane-nat.png) · [Host-only adapter](screenshots/04-control-plane-hostonly.png) · [Static control-plane IP](screenshots/06-control-plane-static-ip.png)

### 2. Configure the container runtime and Kubernetes tooling

I installed/configured **containerd**, then prepared Kubernetes tooling for kubeadm-based cluster administration. Unlike a managed EKS control plane, this setup made the underlying Linux machines and their runtime dependencies part of the work.

**Evidence:** [containerd](screenshots/16-control-plane-containerd.png) · [Kubernetes tools](screenshots/22-kubernetes-tools-installed.png)

### 3. Bootstrap the control plane and join the worker

I initialised the Kubernetes control plane with **kubeadm**, configured administrative access to the API server, joined the worker, and checked node status. On a self-managed cluster, this process involves the relationship between kubeadm, the API server, node agents, the container runtime and networking.

**Observed result:** Two nodes became `Ready`.

**Evidence:** [Two-node cluster Ready](screenshots/49-two-node-kubernetes-cluster-ready.png) · [Kubernetes system Pods](screenshots/43-kubernetes-system-pods-running.png)

### 4. Install the Container Network Interface

I deployed **Calico** for Kubernetes Pod networking and validated the system components and workload connectivity. This was an important part of learning why a cluster can have an API server running yet still be unable to schedule or network application Pods successfully.

**Evidence:** [Calico installation](screenshots/42-calico-cni-installation.png) · [ClusterIP test](screenshots/52-nginx-clusterip-service-connectivity.png)

### What is managed directly in this lab?

| Area | kubeadm homelab responsibility | Contrast with AWS EKS |
| --- | --- | --- |
| **Control plane** | Initialise and troubleshoot Kubernetes control-plane components | AWS manages the EKS control plane |
| **Node OS** | Prepare Ubuntu VMs, networking and machine resources | Managed node groups abstract much of node provisioning |
| **Runtime / CNI** | Configure containerd and install Calico | EKS integrates with AWS-focused cluster tooling/add-ons |
| **Workloads** | Deploy manifests, inspect Pods and manage updates | Still a Kubernetes user/platform responsibility on EKS |
| **Failures** | Diagnose underlying VM performance and cluster access | Managed clusters reduce—but do not eliminate—operational work |

**Repository boundary:** The cluster was bootstrapped **manually**, with the setup documented through notes, screenshots and this README. The repository contains **application YAML**, but **not a complete infrastructure-as-code or automated kubeadm provisioning system**. The deployment guide below assumes an existing, working cluster.

---

## Application Manifests — What the YAML Actually Configures

### NGINX Deployment

The committed [`manifests/deployment.yaml`](manifests/deployment.yaml) specifies:

- **4 desired replicas**, selected with `app: nginx-demo`.
- The `nginx:stable` container image, exposing port `80`.
- A **rolling-update strategy** with `maxSurge: 1` and `maxUnavailable: 0`.
- A **readiness probe** on `/` after an initial five-second delay.
- A **liveness probe** on `/` after an initial ten-second delay.
- Per-container CPU/memory **requests** and **limits**.

Requests help the scheduler make placement decisions; limits constrain container resource consumption. Readiness controls whether a Pod should receive Service traffic; liveness gives Kubernetes a signal for restarting an unhealthy container.

The rolling-update strategy is configured to avoid intentionally reducing available replicas during an update, **subject to adequate cluster capacity and actual probe behaviour**. It is not proof of zero downtime under every failure condition.

### ClusterIP — internal access

[`manifests/service-clusterip.yaml`](manifests/service-clusterip.yaml) creates `nginx-service`:

- Type: `ClusterIP`
- Service port: `80`
- Target port: `80`
- Selector: `app: nginx-demo`

This provides a stable in-cluster Service endpoint for the NGINX Pods.

### NodePort — access from the Windows host

[`manifests/service-nodeport.yaml`](manifests/service-nodeport.yaml) creates `nginx-nodeport`:

- Type: `NodePort`
- Service port / target port: `80` / `80`
- NodePort: `32396`
- Selector: `app: nginx-demo`

The Windows host accessed NGINX through the worker's host-only address at `http://192.168.56.11:32396`.

**Evidence:** [Worker Pod placement](screenshots/51-nginx-workloads-on-worker.png) · [ClusterIP connectivity](screenshots/52-nginx-clusterip-service-connectivity.png) · [NodePort browser access](screenshots/53-nginx-nodeport-browser-access.png)

---

## Kubernetes Operations — Tested Against the Cluster

A working installation is the starting point. I used the running lab to demonstrate **day-to-day workload operations**, not just cluster creation.

| Test | Action | Observed result |
| --- | --- | --- |
| **Scale out** | Scaled `nginx-demo` to four Pods | Four NGINX replicas ran on the worker |
| **Pod self-healing** | Deleted a running NGINX Pod | Deployment/ReplicaSet controller created a replacement |
| **Rolling update** | Changed the application image version | Rollout completed successfully |
| **Post-update check** | Tested the application through the Service | NGINX remained accessible after the rollout |
| **Rollback** | Restored an earlier Deployment revision | Rollback completed; application connectivity rechecked |

### 1. Horizontal scaling — four replicas

```bash
kubectl scale deployment nginx-demo --replicas=4
kubectl get deployment nginx-demo
kubectl get pods -l app=nginx-demo -o wide
```

**Evidence:** [Four-replica scaling result](screenshots/55-nginx-horizontal-scaling-four-replicas.png)

This was **manual scaling** through Kubernetes. It was **not** a Horizontal Pod Autoscaler (HPA), and all four Pods were scheduled on **one worker node**. The committed manifest also records the desired replica count as four.

### 2. Pod self-healing — controller reconciliation

After deleting one NGINX Pod, I observed the ReplicaSet/Deployment controllers replacing it to bring the actual Pod count back to the desired state.

```bash
kubectl get pods -l app=nginx-demo
# Delete an identified NGINX Pod during a controlled test:
kubectl delete pod <nginx-pod-name>
kubectl get pods -l app=nginx-demo -w
```

**Evidence:** [Replacement Pod after deletion](screenshots/54-kubernetes-self-healing-replacement-pod.png)

This verifies **Pod-level reconciliation**. It does not prove that the cluster tolerates losing the only worker node.

### 3. Rolling update — controlled workload change

I performed a Deployment update using a different NGINX image revision, inspected rollout progress, then tested service connectivity.

```bash
kubectl rollout status deployment/nginx-demo
kubectl rollout history deployment/nginx-demo
kubectl get pods -l app=nginx-demo -o wide
```

**Evidence:** [Successful rolling update](screenshots/56-nginx-rolling-update-success.png) · [Application access after update](screenshots/57-nginx-post-update-connectivity.png)

The manifest specifies **`maxSurge: 1` / `maxUnavailable: 0`**, but actual rollout success depends on the worker having sufficient spare capacity to schedule the additional Pod.

### 4. Rollback — restore a previous revision

```bash
kubectl rollout undo deployment/nginx-demo
kubectl rollout status deployment/nginx-demo
kubectl rollout history deployment/nginx-demo
```

**Evidence:** [Application accessible after rollback](screenshots/58-nginx-rollback-connectivity.png)

**Key lesson:** Deployment revisions give an operational recovery option after an unwanted rollout, but rollback should be followed by actual workload and connectivity checks.

> **Configuration consistency:** Imperative operations such as `kubectl scale` or `kubectl rollout undo` change live state. If a change is meant to persist, the Git-tracked YAML should be updated to reflect the intended desired state. This lab did not implement Argo CD or GitHub Actions as a deployment controller.

---

## Troubleshooting & Root-Cause Investigation

The value of this project was the need to diagnose the **Linux and virtualization infrastructure behind Kubernetes**. I documented what I observed, how I investigated it and what the lab showed after recovery.

### Incident 1 — VirtualBox CPU lockups affected cluster bootstrap

**Problem:** During setup, the control-plane VM encountered CPU/performance problems and Kubernetes components failed to progress normally.

**Investigation:** I checked VM resource allocation and host virtualisation settings, examined symptoms during bootstrap and investigated whether native hardware virtualisation was available.

**Recovery:** The VM environment was recovered with native hardware acceleration available, and I continued cluster setup and validation.

**Evidence:** [Native VT-x acceleration confirmation](screenshots/37-native-vtx-acceleration-confirmed.png) · [Cluster subsequently Ready](screenshots/49-two-node-kubernetes-cluster-ready.png)

**Engineering lesson:** Control-plane symptoms can originate below Kubernetes—in the host hypervisor, CPU virtualisation or VM resource configuration. Repeatedly rerunning `kubeadm` is not a substitute for diagnosing that dependency.

### Incident 2 — Kubernetes administrative access and RBAC

**Problem:** Cluster administration and API access required investigation during bootstrap/recovery.

**Investigation:** I examined administrative connectivity and the distinction between **authentication** (who a client is) and **authorisation/RBAC** (what operations that identity is permitted to perform).

**Recovery:** I worked through access recovery and subsequently continued cluster administration, node validation and application deployment.

**Evidence:** [RBAC/access recovery screenshot](screenshots/28-kubernetes-rbac-recovery.png) · [System Pods](screenshots/43-kubernetes-system-pods-running.png)

**Engineering lesson:** Possessing a kubeconfig does not automatically mean every API request is authorised. Permissions, certificates, API availability and client context all matter. This evidence records investigation/recovery; it is **not** proof of a formal RBAC least-privilege audit.

### Incident 3 — Distinguishing Pod redundancy from node availability

**Observation:** Four NGINX replicas were running, but **all four were placed on the same worker**.

**Implication:** Kubernetes can replace a deleted Pod while the node remains healthy; the workload may still become unavailable if that worker fails.

**Next design step:** Add additional worker capacity, spread replicas across nodes and test recovery from a real node failure. This was **not performed** in the current lab.

### Incident 4 — Verifying both Kubernetes Service paths

**Observation:** An in-cluster client could use the **ClusterIP** Service, while the Windows host reached NGINX via the **NodePort** Service and host-only adapter.

**Engineering lesson:** A reachable Pod or a Ready node does not by itself prove the request path. The relevant Service selector, Pod endpoints, target ports, node address and surrounding network connectivity all need to align.

**Evidence:** [ClusterIP test](screenshots/52-nginx-clusterip-service-connectivity.png) · [NodePort browser result](screenshots/53-nginx-nodeport-browser-access.png)

---

## Repository Structure

```text
kubeadm-kubernetes-lab/
├── README.md
├── docs/                         # Supporting project documentation
├── manifests/
│   ├── deployment.yaml
│   ├── service-clusterip.yaml
│   └── service-nodeport.yaml
└── screenshots/                  # 19 deployment evidence images
```

- [`manifests/deployment.yaml`](manifests/deployment.yaml) — desired NGINX replicas, probes, resources and rollout strategy.
- [`manifests/service-clusterip.yaml`](manifests/service-clusterip.yaml) — Kubernetes-internal access.
- [`manifests/service-nodeport.yaml`](manifests/service-nodeport.yaml) — access from the host-only network.
- [`screenshots/`](screenshots/) — captured infrastructure setup, tests, troubleshooting and architecture.

The repository **does not contain automated VirtualBox provisioning or a complete kubeadm bootstrap script**. It is a record of a completed, hands-on cluster lab and a reusable set of **application manifests**.

---

## Deploy the Application to an Existing Cluster

These commands install the NGINX **workload and Services** on an **already functioning** Kubernetes cluster. They do **not** create virtual machines, initialise the control plane or install the CNI.

### Prerequisites

- An accessible Kubernetes cluster with `kubectl` configured.
- Nodes reporting `Ready` and a working CNI/container runtime.
- Permission to create Deployments and Services.
- Sufficient node CPU/memory for the configured four replicas and any surge Pod.
- If accessing via the Windows host, a working host-to-node network path and firewall allowance for the NodePort.

### 1. Clone the repository

```bash
git clone https://github.com/ismaaeelahmed11/kubeadm-kubernetes-lab.git
cd kubeadm-kubernetes-lab
```

### 2. Check cluster health and access

```bash
kubectl config current-context
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods -A
```

Check that the API server responds and that the cluster is healthy enough to schedule workloads. If commands return authorisation errors, investigate kubeconfig identity and RBAC before modifying resources.

### 3. Inspect and validate the manifests

```bash
kubectl apply --dry-run=client -f manifests/
kubectl diff -f manifests/   # Optional: review changes against an existing cluster
```

The Deployment currently uses the mutable `nginx:stable` tag. For repeatable releases, pin a tested image version or digest before deploying to a shared environment.

### 4. Deploy NGINX and Services

```bash
kubectl apply -f manifests/
kubectl rollout status deployment/nginx-demo
```

### 5. Confirm workload and Service state

```bash
kubectl get deployment nginx-demo
kubectl get pods -l app=nginx-demo -o wide
kubectl get services nginx-service nginx-nodeport
kubectl get endpointslices -l kubernetes.io/service-name=nginx-nodeport
```

Verify Pod readiness and Service endpoints rather than relying only on the existence of objects.

### 6. Test access

From the **Windows host** on the original VirtualBox host-only network:

```text
http://192.168.56.11:32396
```

On a different cluster, use a **reachable node address** and inspect the assigned NodePort from `kubectl get svc nginx-nodeport`. The same local address will not necessarily work elsewhere.

### 7. Observe and recover a deployment

```bash
kubectl rollout history deployment/nginx-demo
kubectl rollout status deployment/nginx-demo
# If a subsequent revision needs to be undone:
kubectl rollout undo deployment/nginx-demo
kubectl rollout status deployment/nginx-demo
```

**Important:** An actual rollback requires an earlier Deployment revision. Do not assume a freshly created Deployment already has revision history.

---

## Implementation Evidence — 19 Screenshots

The gallery is organised for technical review. **Four hero screenshots** are shown near the top of the README and linked again here to avoid unnecessarily repeating large images. Every path references an evidence file recorded in the project.

<details>
<summary><strong>1. VM infrastructure & Linux preparation — 5 screenshots</strong></summary>

### VirtualBox NAT adapter

Outbound VM connectivity for package installation and image downloads.

![VirtualBox NAT networking](screenshots/03-control-plane-nat.png)

### VirtualBox host-only adapter

Windows host ↔ Ubuntu VM connectivity for administration and NodePort access.

![Host-only network adapter](screenshots/04-control-plane-hostonly.png)

### Static control-plane IP address

Predictable address configured for the cluster control plane.

![Ubuntu control-plane static IP](screenshots/06-control-plane-static-ip.png)

### containerd configuration

Container runtime configured before running Kubernetes workloads.

![containerd configuration evidence](screenshots/16-control-plane-containerd.png)

### Kubernetes tools installed

Tooling prepared for Kubernetes cluster administration.

![Kubernetes tools installation](screenshots/22-kubernetes-tools-installed.png)

</details>

<details>
<summary><strong>2. Cluster bootstrap, networking & readiness — 3 screenshots</strong></summary>

### Calico CNI installation

Pod network component installed in the self-managed cluster.

![Calico CNI installation](screenshots/42-calico-cni-installation.png)

### Kubernetes system Pods

Cluster-system component status captured during validation.

![Kubernetes system Pods](screenshots/43-kubernetes-system-pods-running.png)

### Two nodes Ready

Control plane and worker joined the cluster and reported Ready.

[View cluster Ready screenshot (featured above)](screenshots/49-two-node-kubernetes-cluster-ready.png)

</details>

<details>
<summary><strong>3. Application deployment & service networking — 3 screenshots</strong></summary>

### NGINX scheduled on worker

Pod placement on the worker node.

![NGINX worker Pod placement](screenshots/51-nginx-workloads-on-worker.png)

### ClusterIP connectivity

In-cluster HTTP Service access demonstrated.

![ClusterIP Service connectivity](screenshots/52-nginx-clusterip-service-connectivity.png)

### NGINX through NodePort from Windows

Browser access through the host-only VM network.

[View NodePort browser screenshot (featured above)](screenshots/53-nginx-nodeport-browser-access.png)

</details>

<details>
<summary><strong>4. Deployment operations — 5 screenshots</strong></summary>

### Pod self-healing

Deleted Pod replaced as the Deployment controller reconciled desired replicas.

[View Pod replacement screenshot (featured above)](screenshots/54-kubernetes-self-healing-replacement-pod.png)

### Four NGINX replicas

Manually scaled Deployment and inspected running replicas.

![NGINX horizontal scaling to four replicas](screenshots/55-nginx-horizontal-scaling-four-replicas.png)

### Rolling update succeeded

Deployment rollout completed.

![NGINX rolling-update result](screenshots/56-nginx-rolling-update-success.png)

### Connectivity after update

HTTP access verified after the update.

![NGINX application connectivity after update](screenshots/57-nginx-post-update-connectivity.png)

### Connectivity after rollback

Previous revision restored and HTTP access rechecked.

[View post-rollback connectivity screenshot (featured above)](screenshots/58-nginx-rollback-connectivity.png)

</details>

<details>
<summary><strong>5. Troubleshooting — 2 screenshots</strong></summary>

### Cluster access / RBAC investigation

Evidence retained from administrative access troubleshooting.

![Kubernetes administrative access and RBAC recovery](screenshots/28-kubernetes-rbac-recovery.png)

### Native virtualization acceleration

Evidence from diagnosing VirtualBox performance and CPU issues.

![VT-x acceleration confirmation](screenshots/37-native-vtx-acceleration-confirmed.png)

</details>

<details>
<summary><strong>6. Architecture — 1 screenshot</strong></summary>

### kubeadm cluster design

Two Ubuntu VMs, control-plane/worker roles, host-only networking and NGINX Service paths.

[View full architecture diagram (featured above)](screenshots/59-kubeadm-kubernetes-architecture.png)

</details>

---

## Operational Boundaries & Next Improvements

| Area | What was demonstrated | Next step for a more resilient platform |
| --- | --- | --- |
| **Control plane** | One kubeadm-managed control plane | Additional control-plane nodes, etcd resilience and tested backups/recovery |
| **Workload availability** | Four replicas on **one worker** | Multiple worker nodes, topology spreading and node-failure testing |
| **Scaling** | Manual `kubectl scale` | Metrics Server, HPA and realistic resource/load tests |
| **Deployments** | Rolling update and rollback | Pinned image digests, automated validation and documented release gates |
| **Networking** | Calico, ClusterIP and local NodePort | NetworkPolicy testing, ingress/TLS and appropriate network segmentation |
| **Security** | Kubernetes administrative access/RBAC troubleshooting | Least-privilege service accounts, RBAC reviews, security admission policies |
| **Provisioning** | Manual VirtualBox and kubeadm setup | Repeatable VM/bootstrap automation with Ansible, Vagrant or scripts |
| **Lifecycle** | kubeadm cluster installation and application operations | **Kubernetes control-plane and node version upgrade** with compatibility checks and rollback planning |
| **Observability** | `kubectl` and system Pod checks during lab operations | Metrics, centralised logging, dashboards, alerts and runbooks |
| **Recovery** | Pod-level replacement and workload rollback | Test actual node loss, etcd restoration and disaster recovery |

**Important:** A full **kubeadm version upgrade**—including control plane, kubelet and kubectl—**was not performed** in this project. It is a logical **future cluster-lifecycle exercise**, not an achievement listed as completed. Likewise, I did not validate automatic worker-node failover or a highly available control plane.

---

## What This Project Demonstrates

This project moved me beyond deploying applications onto an AWS-managed cluster and into the Linux and Kubernetes infrastructure that supports them.

I **prepared nodes, configured networking and containerd, bootstrapped Kubernetes, installed Calico, joined a worker, deployed YAML-defined workloads, exposed Services, performed controlled application operations and troubleshot setup failures**. The supporting manifests and screenshots let a reviewer verify specific technical claims.

The key engineering lesson was that **Kubernetes workload health depends on several layers working together**: the host VM, Linux networking, container runtime, CNI, API access, scheduling, controllers and Service routing. A successful Pod deployment is useful; understanding what breaks underneath it is more valuable.

**Project context:** This is independent practical portfolio work completed after my [AWS EKS engineering project](https://github.com/ismaaeelahmed11/eks-project), alongside my DevOps training and portfolio development in 2026. It is **not represented as paid commercial Kubernetes administration experience**.

### Related Projects

- **[AWS EKS — Terraform, Kubernetes, GitOps, CI/CD & Observability](https://github.com/ismaaeelahmed11/eks-project)** — AWS-managed Kubernetes platform with NGINX Ingress, Argo CD, cert-manager, ExternalDNS and Prometheus/Grafana. [Watch walkthrough](https://youtu.be/RDmnaTi2auE).
- **[AWS ECS Fargate — Terraform, Docker, GitHub Actions & HTTPS](https://github.com/ismaaeelahmed11/devops-learning/tree/main/ecs-project)** — containerised Go service deployed through AWS load balancing and infrastructure automation. [Watch walkthrough](https://youtu.be/qpIZlJkWmos).
- **[DevOps Engineering Portfolio](https://github.com/ismaaeelahmed11)** — Linux, Bash, Docker, Terraform, AWS, Kubernetes and delivery projects.

---

**Ismaaeel Ahmed — DevOps & Cloud Engineering**  
[GitHub](https://github.com/ismaaeelahmed11) · [LinkedIn](https://www.linkedin.com/in/ismaaeelahmed) · [Project walkthrough](https://youtu.be/a9hATioCGcs)
