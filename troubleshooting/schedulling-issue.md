#  Kubernetes Scheduling & Upgrade Simulation  
## DevOps Case Study (Interview-Ready Documentation)

---

##  Overview

This project simulates real-world Kubernetes production scenarios to demonstrate how scheduling decisions, resource constraints, and cluster upgrades impact application availability.

It focuses on practical DevOps skills such as:

- Kubernetes scheduling behavior
- Cluster upgrade simulation (node draining)
- Handling scheduling constraints
- Debugging production incidents
- Resource optimization
- Reliability engineering (PDB, affinity, taints)

---

##  Objective

To understand and demonstrate how Kubernetes behaves under real operational conditions, especially:

- Why pods fail to schedule
- Why applications become unavailable during upgrades
- How resource misconfiguration leads to performance issues
- How to debug production-like Kubernetes incidents

---

##  Architecture Setup

- Kubernetes Cluster (K3s / EKS-like simulation)
- 1 Control Plane Node
- Multiple Worker Nodes
- Sample microservices deployed across nodes

---

##  Core Engineering Concepts Tested

---

## 1️ Kubernetes Scheduling Model

Kubernetes schedules pods based on:

- Node availability
- Resource requests (CPU/Memory)
- Node labels
- Taints and tolerations
- Affinity rules

### Default Behavior

Control plane nodes are tainted:

```bash
node-role.kubernetes.io/control-plane=true:NoSchedule
```
This ensures workloads do not run on control plane nodes.

### 2 Taints & Tolerations (Access Control for Nodes)

Scenario

We isolate a node for backend workloads only.

```bash
kubectl taint nodes dedicated=backend:NoSchedule
```

***Problem***
Pods without tolerations are blocked from scheduling.

***Fix**

```YAML
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "backend"
    effect: "NoSchedule"
```

### DevOps Insight

Taints act like access control for scheduling

### 3. Node Selector Constraints (Hard Placement Rules)

```YAML
nodeSelector:
  role: worker1
```

***Failure Scenario***

Pods become unschedulable when:

* ***Node is drained***
* ***Label does not exist***
* ***Cluster scales incorrectly***

***Insight***

NodeSelector is rigid and can cause availability risks.

# 🔄 Upgrade Simulation (Node Drain Scenario)

## ⌨️ Command Used
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data


What Happens Internally
Node is cordoned: No new pods can be scheduled to the node.

Running pods are evicted: Existing pods are gracefully terminated.

Scheduler reassigns: The Control Plane attempts to reschedule pods to other available nodes.

⚠️ Real Production Issues Observed
1. PDB Blocking Node Drain
The system cannot evict a pod because doing so would violate the PodDisruptionBudget.

Impact: Upgrade is blocked; node remains in an unschedulable state.

2. Pods Stuck in Pending State
Root Causes:

Taints without corresponding tolerations.

nodeSelector or Affinity mismatches.

Insufficient CPU / Memory on remaining nodes.

🛠️ Incident Debugging Approach
Step 1: Cluster Visibility
Bash
kubectl get pods -o wide
Step 2: Deep Pod Inspection
Bash
kubectl describe pod <pod-name>
Focus on: The Events section and specific Scheduling messages.

Step 3: Error Interpretation
Error Type	Root Cause
Insufficient CPU	Resource exhaustion
Untolerated taint	Node access restriction
NodeSelector mismatch	No valid node match
PDB violation	Safety policy blocking eviction
📊 Resource Optimization Analysis
Observed Issue: Over-Provisioning
Metric	Value
CPU Request	200m
Actual Usage	20m
Result: 👉 10x resource wastage

✅ Fix Strategy
Gradually adjust requests/limits.

Monitor performance under load.

Avoid aggressive tuning in production.

📈 Scaling Strategy
When the cluster is under pressure:

Option 1 (Preferred): Add nodes via Cluster Autoscaler / ASG.

Option 2: Reduce resource requests.

🚨 Performance Debugging (Real Production Scenario)
If pods are running but system is slow, check:

CPU / memory utilization.

Application logs.

Inter-service latency.

Database performance bottlenecks.

🎯 Key DevOps Learnings
Scheduling is constraint-driven: Conflicting constraints can prevent pod placement.

Upgrades expose hidden issues: Node draining reveals misconfigured deployments.

Resource requests matter more than usage: Over-provisioning wastes cluster capacity.

PDB improves reliability but reduces flexibility: Too strict PDB can block operations.

Bottlenecks shift during scaling: Fixing one layer may expose another (e.g., database).

🧾 Conclusion
This simulation demonstrates how Kubernetes behaves under real production conditions. It highlights the importance of:

Proper scheduling design

Resource tuning

Upgrade planning

System observability

Incident debugging methodology

🧑‍💻 Author Notes (Interview Framing)
This project reflects hands-on experience with:

Production-grade Kubernetes operations

Incident response workflows

Cluster reliability engineering

Capacity planning and optimization
