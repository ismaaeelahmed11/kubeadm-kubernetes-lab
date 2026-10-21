# Self-Managed Kubernetes Cluster with kubeadm

## Overview

A hands-on Kubernetes homelab built using two Ubuntu virtual machines,
kubeadm, containerd and Calico networking.

The project demonstrates cluster administration, application deployment,
service networking, scaling, self-healing and rolling updates.

## Architecture

| Node | Role | IP address |
|---|---|---|
| k8s-control | Control plane | 192.168.56.10 |
| k8s-worker | Worker node | 192.168.56.11 |

- Kubernetes: v1.35.9
- Container runtime: containerd 2.2.2
- CNI: Calico
- Application: NGINX

## Features

- Two-node self-managed Kubernetes cluster provisioned with kubeadm
- Calico pod networking
- Declarative Deployment and Service manifests
- Four NGINX replicas on the worker node
- ClusterIP and NodePort Services
- CPU and memory requests and limits
- Readiness and liveness probes
- Rolling updates and rollbacks
- Pod self-healing and manual horizontal scaling

## Repository Structure

```text
kubeadm-kubernetes-lab/
├── README.md
└── manifests/
    ├── deployment.yaml
    ├── service-clusterip.yaml
    └── service-nodeport.yaml
```

## Deploy the Application

Apply the manifests:

```bash
kubectl apply -f manifests/
```

Check the Deployment and Services:

```bash
kubectl get nodes -o wide
kubectl get deployments
kubectl get pods -o wide
kubectl get services
```

Check rollout status:

```bash
kubectl rollout status deployment/nginx-demo
```

## Access NGINX

The NodePort Service uses port 32396.

From a machine that can reach the worker's host-only network:

```text
http://192.168.56.11:32396
```

## Scaling and Recovery

Scale to four replicas:

```bash
kubectl scale deployment nginx-demo --replicas=4
```

Inspect rollout history:

```bash
kubectl rollout history deployment/nginx-demo
```

Roll back to the previous revision:

```bash
kubectl rollout undo deployment/nginx-demo
```

## Validation

The manifests can be checked without changing live resources:

```bash
kubectl apply --dry-run=client -f manifests/
```

## Limitations

This is a local virtual-machine lab, not a production deployment.
All application replicas currently run on one worker node, so they do
not provide resilience against failure of that worker.

The cluster IP addresses and NodePort URL are specific to this lab
environment and may need changing in another environment.
