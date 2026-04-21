# 🚀 Scenario-Based DevOps Project: Cost Optimization with k3s for Staging

## 📌 Project Scenario

In a simulated enterprise environment, both **production** and **staging** applications were running on **Amazon EKS** clusters.

Although EKS provided a reliable managed Kubernetes platform, running separate clusters for non-production workloads significantly increased monthly cloud costs.

Management requested a more cost-effective solution for the **staging environment**, while keeping **production on EKS** for stability, scalability, and managed operations.

---

## 🎯 Objective

My responsibility was to design and implement a new staging environment that would:

* Reduce infrastructure cost
* Maintain Kubernetes compatibility with production
* Support application testing before releases
* Keep deployment workflows consistent across environments

---

## 💡 Proposed Solution

I recommended a **hybrid Kubernetes model**:

### Production Environment

* Remain on **Amazon EKS**
* Continue using managed Kubernetes for live workloads

### Staging Environment

* Migrate to a **self-managed k3s cluster** hosted on AWS EC2

This provided a lightweight and lower-cost Kubernetes environment while preserving standard Kubernetes functionality.

---

## 🏗️ Implementation

I deployed a **multi-node k3s cluster** consisting of:

* 1 Control Plane Node
* 2 Worker Nodes

### Components Configured

* Flannel CNI for pod networking
* CoreDNS for internal service discovery
* NodePort services for external staging access
* Containerd runtime

### Security Rules Opened

* TCP 6443 – Kubernetes API Server
* TCP 10250 – Kubelet
* UDP 8472 – Flannel VXLAN
* TCP 30000–32767 – NodePort Services
* TCP 22 – SSH Administration

---

## 🧪 Validation Performed

After deployment, I validated the environment by testing:

✔ Worker nodes successfully joined cluster
✔ Pods scheduled across multiple nodes
✔ Pod-to-pod communication across nodes
✔ Internal DNS resolution via CoreDNS
✔ Service access using ClusterIP and NodePort
✔ Standard Kubernetes manifests deployed successfully

---

## 📉 Result

The staging environment was successfully moved to k3s, which:

* Reduced operating cost compared to EKS
* Preserved Kubernetes deployment compatibility
* Enabled continued QA/UAT testing workflows
* Maintained production stability on EKS

---

## 🧠 Skills Demonstrated

* Kubernetes Administration
* k3s Cluster Setup
* AWS EC2 & Security Groups
* Linux Administration
* Networking Troubleshooting
* DNS / CoreDNS Debugging
* Cost Optimization Strategy
* Environment Migration Planning

---

## 🎤 Interview Summary

If asked about this project in an interview, I would summarize it as:

