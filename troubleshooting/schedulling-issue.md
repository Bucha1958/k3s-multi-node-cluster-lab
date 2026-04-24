☸️ Kubernetes Scheduling & Upgrade SimulationDevOps Case Study: Interview-Ready Documentation📖 OverviewThis project simulates real-world Kubernetes production scenarios to demonstrate how scheduling decisions, resource constraints, and cluster upgrades impact application availability.It serves as a practical demonstration of high-level DevOps competencies, including:Kubernetes Scheduling: Advanced pod placement logic.Lifecycle Management: Cluster upgrade simulations and node draining.Reliability Engineering: Implementing PDBs, affinity rules, and taints.Incident Response: Debugging production-style scheduling failures.Efficiency: Resource optimization and capacity planning.🎯 ObjectivesTo analyze and demonstrate Kubernetes behavior under operational stress, specifically:Identifying why pods fail to schedule (Pending states).Ensuring application uptime during maintenance/upgrades.Mapping the impact of resource misconfiguration on cluster performance.Standardizing a debugging workflow for cluster incidents.🏗️ Architecture SetupCluster: K3s / EKS-like simulation.Control Plane: 1 Dedicated Node (Tainted).Workers: Multiple Worker Nodes with varying labels.Workloads: Sample microservices with diverse scheduling requirements.🧠 Core Engineering Concepts1. Kubernetes Scheduling ModelThe scheduler assigns pods based on a filter-and-score mechanism considering:Resource requests (CPU/Memory)Node labels & SelectorsTaints/Tolerations and Affinity/Anti-affinity rules.Default Behavior: Control plane nodes are typically protected by the following taint to prevent user workloads from consuming system resources:Bashnode-role.kubernetes.io/control-plane=true:NoSchedule
2. Taints & Tolerations (Access Control)Scenario: Isolating a node specifically for backend workloads.Action: kubectl taint nodes <node-name> dedicated=backend:NoScheduleProblem: Standard pods remain "Pending" as they lack permission to run there.Fix:YAMLtolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "backend"
    effect: "NoSchedule"
💡 DevOps Insight: Taints act as "keep away" rules, while tolerations are the "keys" to enter.3. Node Selector Constraints (Hard Rules)Using nodeSelector for rigid placement (e.g., role: worker1).Risk: If the specific node is drained or the label is missing, the pod cannot failover to other healthy nodes.Insight: nodeSelector is simple but lacks the flexibility of Affinity rules, often leading to availability risks during scaling.4. Pod Anti-Affinity (High Availability)Ensuring replicas are distributed across different physical hosts to avoid single points of failure.YAMLaffinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: frontend
          topologyKey: kubernetes.io/hostname
5. Pod Disruption Budget (PDB)PDBs protect application availability during voluntary disruptions (e.g., node drains for upgrades).YAMLapiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: frontend
⚠️ Trade-off: A PDB that is too strict (e.g., minAvailable equals total replicas) can physically block a cluster upgrade from proceeding.🔄 Upgrade & Drain SimulationCommand:Bashkubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
Observed Production IssuesPDB Blocking: The drain hangs because evicting a pod would violate the minimum availability threshold.Pending Traps: Evicted pods cannot find a new home because of mismatched Taints or insufficient CPU on remaining nodes.🛠️ Incident Debugging MethodologyStep 1: Cluster VisibilityBashkubectl get pods -o wide
kubectl get nodes --show-labels
Step 2: Deep InspectionBashkubectl describe pod <pod-name>
Check: The "Events" section usually contains the specific reason for scheduling failure (e.g., 0/3 nodes are available: 1 node(s) had untolerated taint).Step 3: Error MatrixError TypeRoot CauseInsufficient CPUResource exhaustion / Over-provisioningUntolerated TaintNode access restrictionNodeSelector MismatchIncorrect labels or rigid placementPDB ViolationSafety policy blocking maintenance📊 Resource Optimization AnalysisObserved Issue: Over-provisioning.Metric: CPU Request = 200m | Actual Usage = 20m.Result: 10x resource wastage.Strategy: Implement Vertical Pod Autoscaler (VPA) or manually tune requests based on historical Prometheus metrics to increase bin-packing efficiency.📈 Key LearningsScheduling is Constraint-Driven: Overlapping rules (Affinity + Taints + Requests) can easily create "deadlocks."Upgrades Reveal Weakness: If a node drain fails, your HA configuration is likely flawed.Requests > Usage: The scheduler only cares about what you request, not what you actually use.Observability is Mandatory: Without events and logs, Kubernetes scheduling is a "black box."