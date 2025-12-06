---
layout: default
title: Setting up k3s + AWX on Ubuntu (Master + Worker Nodes)
date: 2025-12-04 06:01:42 -0500
categories: [Python, DevOps, Code, Programming]
tags: [kubernetes, k3s, awx, ansible, helm, ubuntu]
excerpt: "Install k3s on two Ubuntu nodes (master + worker) and deploy AWX using the AWX Operator with Helm."
---

# 🚀 Setting up k3s + AWX on Ubuntu (Master + Worker Nodes)

This guide walks through installing a lightweight k3s Kubernetes cluster (1 master + 1 worker) on Ubuntu and deploying AWX (Ansible Tower) using the AWX Operator and Helm.

Table of contents
- [Prerequisites and System Setup](#prerequisites-and-system-setup)
- [Install k3s (Master)](#install-k3s-master)
- [Install k3s (Worker)](#install-k3s-worker)
- [Fix kubectl Permissions](#fix-kubectl-permissions)
- [Verify Cluster](#verify-cluster)
- [Install AWX Operator (Helm)](#install-awx-operator-helm)
- [Deploy AWX Instance](#deploy-awx-instance)
- [Access AWX UI](#access-awx-ui)
- [Troubleshooting & Notes](#troubleshooting--notes)

---

## Prerequisites and System Setup {#prerequisites-and-system-setup}

These steps should be done on both the master and worker nodes.

System requirements
- OS: Ubuntu 20.04 or 22.04
- Minimum resources: 1 CPU, 2 GB RAM (AWX and cluster benefit from more)
- Connectivity: Internet access
- Optional: static IPs recommended for master and worker

Disable swap (required for Kubernetes components):
```bash
sudo swapoff -a
# To make permanent, remove any swap entry from /etc/fstab
```

Update and install basic dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget ca-certificates gnupg lsb-release
```

### Configure Firewall (UFW)

Open required ports (run on nodes where UFW is enabled). Adjust rules to your environment — if you plan to use NodePort ranges or an ingress, open those as needed.

```bash
sudo ufw allow OpenSSH
# Kubernetes API server (master)
sudo ufw allow 6443/tcp
# Kubelet
sudo ufw allow 10250/tcp
# flannel VXLAN (k3s default network) - needed if using flannel
sudo ufw allow 8472/udp
# NodePort range (default)
sudo ufw allow 30000:32767/tcp

sudo ufw reload
```

Note: If UFW blocks forwarded packets, follow the UFW docs to allow forwarding (k3s modifies iptables but make sure forwarding is permitted).

---

## Install k3s (Master) {#install-k3s-master}

Run the following only on the master node.

Install k3s (default channel):
```bash
curl -sfL https://get.k3s.io | sh -
```

Or pin a version:
```bash
# Example: install a specific k3s version
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.27.11+k3s1" sh -
```

Verify service and nodes:
```bash
sudo systemctl status k3s
sudo kubectl get nodes
```

Get the node join token for worker nodes:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token
# copy this value to use when joining workers
```

---

## Install k3s (Worker) {#install-k3s-worker}

Run on the worker node(s). Replace MASTER_IP and NODE_TOKEN with values from the master.

```bash
curl -sfL https://get.k3s.io | K3S_URL="https://MASTER_IP:6443" K3S_TOKEN="NODE_TOKEN" sh -
```

Verify the agent:
```bash
sudo systemctl status k3s-agent
```

---

## Fix kubectl Permissions (Master Node) {#fix-kubectl-permissions}

If you see permission issues for /etc/rancher/k3s/k3s.yaml, copy the kubeconfig to your user and fix ownership:

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
# If accessing from a different machine, change server address inside ~/.kube/config
kubectl get nodes
```

Note: When copying to another host, edit the server address in ~/.kube/config to point to your master's IP (by default it may be 127.0.0.1).

---

## Verify Cluster {#verify-cluster}

On the master:
```bash
kubectl get nodes
kubectl get pods -A
```

You should see both master and worker with STATUS=Ready and core system pods (coredns, local-path-provisioner, etc.) running.

---

## Install AWX Operator using Helm {#install-awx-operator-helm}

AWX Operator manages AWX lifecycle. You can install via a Helm chart or by applying manifests.

Install Helm (if not already installed):
```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Create a namespace for AWX:
```bash
kubectl create namespace awx
```

Option A — Install from local chart (if you downloaded the awx-operator chart):
```bash
# assume you downloaded and extracted chart into awx-operator-<version>
cd awx-operator-2.19.1
helm install awx-operator . -n awx
```

Option B — Use upstream manifests or a Helm repo (preferred if a maintained repo exists):
- Check the AWX Operator project's README for the recommended installation method for your operator version.
- Ensure CRDs are installed before applying CRs.

Verify the operator pod:
```bash
kubectl get pods -n awx
# wait for awx-operator-controller-manager to be Running
```

---

## Deploy AWX Instance {#deploy-awx-instance}

Create an AWX custom resource. This example uses NodePort for quick access; for production use LoadBalancer or Ingress with TLS.

Create a file named awx.yaml with:
```yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
  # Optionally configure resources, ingress, or external databases here
```

Apply the custom resource:
```bash
kubectl apply -f awx.yaml -n awx
kubectl get pods -n awx --watch
```

The operator will create the following AWX pods (names may vary):
- awx-web-*
- awx-task-*
- awx-postgres-*
- awx-ee-*

Wait for them to reach Running. You can check pod logs if something fails:
```bash
kubectl logs -n awx deploy/awx-operator-controller-manager
kubectl describe pod <pod-name> -n awx
```

Storage note: AWX requires persistent storage (Postgres PV). k3s provides a default local-path storage class by default; in production use a proper persistent storage class.

---

## Access AWX UI {#access-awx-ui}

Find the NodePort exposing AWX (lookup the service):
```bash
kubectl get svc -n awx
# Look for the awx service and the NodePort mapping, e.g. 80:XXXXX/TCP
```

Get initial admin password:
```bash
kubectl get secret awx-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode && echo
```

Open your browser:
URL: http://<NODE-IP>:<NodePort>
User: admin
Password: (value retrieved above)

Notes:
- For production, prefer exposing AWX via Ingress with TLS or a LoadBalancer.
- Consider changing the admin password immediately after first login.

---

## Troubleshooting & Notes {#troubleshooting--notes}

- If pods are Pending, check PVCs and storage class:
  ```bash
  kubectl get pvc -n awx
  kubectl get sc
  ```
- If AWX web/task pods crashloop, inspect logs:
  ```bash
  kubectl logs -n awx <pod-name>
  kubectl describe pod <pod-name> -n awx
  ```
- If you copied kubeconfig to another host and kubectl can't reach the cluster, edit ~/.kube/config and replace server: https://127.0.0.1:6443 with server: https://MASTER_IP:6443.
- Consider resource sizing: AWX can be memory-heavy; 4+ GB RAM recommended for a comfortable experience.
- Security: Do not expose AWX on NodePort in production. Use ingress with TLS and restrict admin access.

Further reading and official resources
- k3s: https://k3s.io
- AWX Operator: https://github.com/ansible/awx-operator
- AWX project: https://github.com/ansible/awx

---

✅ Summary

You should now have:
- A small k3s cluster (master + worker)
- AWX Operator installed in the awx namespace
- An AWX instance managed by the operator and accessible via NodePort (or Ingress if configured)

If you want, I can:
- Convert this into a ready-to-commit file for your repo,
- Add an example awx.yaml with resource limits, or
- Add an Ingress + TLS example (Let's Encrypt) for secure access.

Which would you like next?
