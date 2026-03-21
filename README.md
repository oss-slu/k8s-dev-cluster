# OSS-SLU Kubernetes Cluster

4-node Kubernetes cluster automation using Ansible.

## Nodes
- oss01 - Control Plane (10.60.63.1)
- oss02 - Worker (10.60.63.2)
- oss03 - Worker (10.60.63.5)
- oss04 - Worker (10.60.63.6)

## Stack
- Kubernetes 1.32
- Flannel CNI
- Prometheus + Grafana
- ArgoCD
- CloudNativePG
- Nginx Ingress

## Usage
ansible-playbook site.yml -i inventory.ini
