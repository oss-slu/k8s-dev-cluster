# OSS-SLU Kubernetes Cluster

Authoritative 4-node bare-metal Kubernetes cluster automation using Ansible.

## Nodes & Inventory Configuration
- **oss01**: Control Plane (`10.60.63.1`, `controlnode`, local connection)
- **oss02**: Worker Node 1 (`10.60.63.2`, `workernode1`, sudo)
- **oss05**: Worker Node 2 (`10.60.63.5`, `oss05`, sudo)
- **oss04**: Worker Node 3 (`10.60.63.6`, `workernode3`, sudo)

## Stack Specifications
- **Kubernetes**: 1.32 (`pkgs.k8s.io/core:/stable:/v1.32`)
- **Container Runtime**: containerd 2.2.1 + containernetworking/plugins v1.4.0
- **CNI**: Calico CNI v3.27.0 (full CNI + IPAM, `10.244.0.0/16` pod CIDR)
- **Monitoring**: Kube-Prometheus-Stack with dynamic sidecar dashboard loading and local-path PV storage
- **GitOps**: Argo CD (all application manifests synchronized via `k8s-manifests`)
- **Ingress & Database**: Ingress-NGINX controller, CloudNativePG operator

## Safety Guards
- **Host Assertion**: Pre-flight assertion in `roles/common` ensures target nodes strictly match inventory hostnames, eliminating cross-node execution hazards.

## Usage & Verification
Ping all nodes:
```bash
ansible -i inventory.ini all -m ping
```

Run cluster playbook:
```bash
ansible-playbook site.yml -i inventory.ini
```
