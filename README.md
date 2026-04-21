# 🚀 K3s Multi-Node Kubernetes Cluster on AWS EC2

## 📌 Overview
This project demonstrates a production-style multi-node Kubernetes cluster built using **k3s** on AWS EC2 instances. It includes a control plane and multiple worker nodes with full networking, DNS, and service discovery validation.

---

## 🏗️ Architecture

- 1 Control Plane (k3s server)
- 2 Worker Nodes (k3s agents)
- Flannel CNI (VXLAN networking - UDP 8472)
- CoreDNS (Cluster DNS: 10.43.0.10)
- Traefik Ingress Controller
- NodePort services for external access

---

## ⚙️ Tech Stack

- Kubernetes (k3s)
- Flannel CNI
- CoreDNS
- AWS EC2 (Ubuntu)
- Containerd runtime

---

## 🚀 What Was Implemented

- Multi-node Kubernetes cluster setup
- Node joining via k3s agent
- Pod deployment across nodes
- Service exposure using NodePort
- Cluster DNS resolution (CoreDNS)
- Pod-to-pod communication across nodes
- Security group configuration for Kubernetes networking

---

## 🧪 Validation Performed

- Pod-to-pod communication across nodes
- Service access via ClusterIP and NodePort
- DNS resolution inside pods
- Cross-node networking validation using Flannel VXLAN

---

## 🧯 Issues Encountered & Fixes

- DNS NXDOMAIN → Fixed by restarting CoreDNS and validating cluster DNS
- NodePort access issues → Fixed by opening AWS Security Group ports (30000–32767)
- Cross-node traffic issues → Fixed by allowing UDP 8472 (Flannel VXLAN)

---

## 📊 Final Result

A fully working multi-node Kubernetes cluster with:
- Working DNS
- Cross-node pod communication
- Service discovery
- External access via NodePort

---
