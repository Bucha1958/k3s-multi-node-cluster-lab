# Kubernetes Cluster Architecture


[ Worker Node 1 ]        [ Worker Node 2 ]
     |                         |
     |------ Flannel VXLAN -----|
               UDP 8472
                     |
              [ Control Plane ]
              - API Server (6443)
              - CoreDNS (10.43.0.10)
              - kube-proxy



---



