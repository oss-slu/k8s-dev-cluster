# OSS-SLU Kubernetes Cluster

4-node Kubernetes cluster automation using Ansible.

## Nodes
- oss01 - Control Plane 
- oss02 - Worker
- oss03 - Worker 
- oss04 - Worker

## Stack
- Kubernetes 1.32
- Flannel CNI
- Prometheus + Grafana
- ArgoCD
- CloudNativePG
- Nginx Ingress

## Usage
ansible-playbook site.yml -i inventory.ini
