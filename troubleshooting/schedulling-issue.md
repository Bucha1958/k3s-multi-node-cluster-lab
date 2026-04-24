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

This ensures workloads do not run on control plane nodes.

2️⃣ Taints & Tolerations (Access Control for Nodes)
Scenario

We isolate a node for backend workloads only.

kubectl taint nodes dedicated=backend:NoSchedule
Problem

Pods without tolerations are blocked from scheduling.

Fix
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "backend"
    effect: "NoSchedule"
DevOps Insight

👉 Taints act like access control for scheduling

3️⃣ Node Selector Constraints (Hard Placement Rules)
nodeSelector:
  role: worker1
Failure Scenario

Pods become unschedulable when:

Node is drained
Label does not exist
Cluster scales incorrectly
Insight

👉 NodeSelector is rigid and can cause availability risks.

4️⃣ Pod Anti-Affinity (High Availability Design)

Used to distribute pods across nodes to avoid single-node failure impact.

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: frontend
          topologyKey: kubernetes.io/hostname
Benefit
Improves resilience
Prevents multiple replicas on same node
5️⃣ Pod Disruption Budget (PDB) – Availability Protection
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: frontend
Purpose

Ensures minimum number of pods remain running during:

Node upgrades
Draining operations
Maintenance events
Trade-off
Can block cluster upgrades if too strict
🔄 Upgrade Simulation (Node Drain Scenario)
Command Used
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
What Happens Internally
Node is cordoned (no new pods scheduled)
Running pods are evicted
Scheduler reassigns pods to other nodes
⚠️ Real Production Issues Observed
1. PDB Blocking Node Drain
Cannot evict pod as it would violate the pod's disruption budget
Impact
Upgrade blocked
Node remains unschedulable
2. Pods Stuck in Pending State
Root Causes
Taints without tolerations
NodeSelector mismatch
Insufficient CPU / Memory
🛠️ Incident Debugging Approach
Step 1: Cluster Visibility
kubectl get pods -o wide
Step 2: Deep Pod Inspection
kubectl describe pod <pod-name>

Focus on:

Events section
Scheduling messages
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
Result

👉 10x resource wastage

Fix Strategy
Gradually adjust requests/limits
Monitor performance under load
Avoid aggressive tuning in production
📈 Scaling Strategy
When cluster is under pressure:
Option 1 (Preferred)
Add nodes via Cluster Autoscaler / ASG
Option 2
Reduce resource requests
🚨 Performance Debugging (Real Production Scenario)

If pods are running but system is slow:

Check:

CPU / memory utilization
Application logs
Inter-service latency
Database performance bottlenecks
🎯 Key DevOps Learnings
1. Scheduling is constraint-driven

Conflicting constraints can prevent pod placement.

2. Upgrades expose hidden issues

Node draining reveals misconfigured deployments.

3. Resource requests matter more than usage

Over-provisioning wastes cluster capacity.

4. PDB improves reliability but reduces flexibility

Too strict PDB can block operations.

5. Bottlenecks shift during scaling

Fixing one layer may expose another (e.g., database).

🧾 Conclusion

This simulation demonstrates how Kubernetes behaves under real production conditions.

It highlights the importance of:

Proper scheduling design
Resource tuning
Upgrade planning
System observability
Incident debugging methodology