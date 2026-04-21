## `01-prerequisites.md`

```md
- AWS EC2 instances (Ubuntu 22.04)
- Security Groups configured:
  - 22 (SSH)
  - 6443 (Kubernetes API)
  - 10250 (kubelet)
  - 30000–32767 (NodePort)
  - 8472 (Flannel VXLAN)
