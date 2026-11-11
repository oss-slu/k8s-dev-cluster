# OSS-SLU Kubernetes Cluster

4-node Kubernetes cluster automation using Ansible.

## Nodes
- oss01 - Control Plane 
- oss02 - Worker
- oss05 - Worker
- oss04 - Worker

## Stack
- Kubernetes 1.32
- Calico CNI (full CNI + IPAM, not just NetworkPolicy - see
  docs/software-baseline-audit.md for why this replaces the "Flannel
  CNI" this README used to claim)
- Prometheus + Grafana
- ArgoCD
- CloudNativePG
- Nginx Ingress

## Usage
ansible-playbook site.yml -i inventory.ini
