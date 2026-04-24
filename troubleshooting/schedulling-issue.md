Kubernetes Scheduling & Upgrade Simulation (Documentation)
Project Overview

This project simulates real-world Kubernetes operational scenarios including:

Pod scheduling behavior
Taints and tolerations
Node affinity and selectors
Pod Disruption Budgets (PDB)
Node draining (upgrade simulation)
Resource constraints and scaling
Debugging production incidents
 Environment Setup
Kubernetes cluster (K3s / EKS-like simulation)
Multi-node cluster:
Control plane node
Worker nodes
 Key Concepts Practiced
1️ Default Scheduling Behavior
Pods are scheduled on any available node unless restricted

Control plane is tainted by default:

node-role.kubernetes.io/control-plane=true:NoSchedule
2️ Taints & Tolerations
Example Taint:
kubectl taint nodes <node> dedicated=backend:NoSchedule
Effect:
Pods without toleration cannot be scheduled
Fix Options:
Remove taint:
kubectl taint nodes <node> dedicated=backend:NoSchedule-
OR add toleration in pod spec
3️ Node Selector Constraint
nodeSelector:
  role: worker1
Issue:
Pods become unschedulable if:
Node is drained
Label doesn’t exist elsewhere
4️ Pod Anti-Affinity

Used to spread pods across nodes:

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchLabels:
            app: frontend
        topologyKey: kubernetes.io/hostname
5️ Pod Disruption Budget (PDB)
Example:
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: frontend
Behavior:
Prevents too many pods from being evicted
Can block kubectl drain
 Upgrade Simulation (Node Drain)
Command:
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
What Happens:
Node is cordoned
Pods are evicted
Scheduler reschedules pods elsewhere
 Issues Encountered
 PDB Blocking Eviction
Cannot evict pod as it would violate the pod's disruption budget
 Pods Stuck Pending

Causes:

Taints
NodeSelector
Insufficient CPU
 Debugging Approach
Step 1: Check Pods
kubectl get pods -o wide
Step 2: Describe Pod
kubectl describe pod <pod>

Focus on:

Events section
Step 3: Common Errors
Error	Meaning
Insufficient cpu	No available resources
Untolerated taint	Node blocked
NodeSelector mismatch	No valid node
PDB violation	Cannot evict
 Resource Optimization
Check usage:
kubectl top pods
Compare with requests:
kubectl describe pod <pod>
 Over-Provisioning Example
Metric	Value
Request CPU	200m
Actual Usage	20m

 Waste = 10x

Fix Strategy
Gradually reduce requests
Monitor performance
Avoid aggressive cuts
 Scaling Strategy
When CPU is insufficient:
Option 1 (Preferred):
Add nodes (Cluster Autoscaler / ASG)
Option 2:
Reduce resource requests
 Runtime Performance Debugging

If pods are running but app is slow:

Check:
CPU / memory usage
Logs
Inter-service latency
Database performance
 Real Lessons Learned
Scheduling constraints can conflict
Draining exposes bad configurations
Requests ≠ actual usage
PDB protects uptime but can block operations
Scaling can shift bottlenecks (e.g., to DB)