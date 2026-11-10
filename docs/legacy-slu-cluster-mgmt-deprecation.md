# Deprecating `slu-cluster-mgmt` (oss_infra_ops#152)

## Read this first: a live security issue, filed separately

While reviewing `slu-cluster-mgmt`'s content for this consolidation, a
**committed root SSH private key** for `oss01` was found
(`ansible/keys`, ~8 months old). Filed as its own, higher-priority
issue: **oss_infra_ops#165**. Rotate that key before anything else in
this document - don't let the deprecation work below be the reason it
sits unaddressed.

## What `slu-cluster-mgmt` actually is

Cloned and reviewed directly (not available in this sandbox otherwise).
It's a **different, older, incompatible Ansible structure** than this
repo, built for a **2-node** topology (`oss01` "Master", `oss02`
"Worker" - its own README says so explicitly) and the
`CoreDesk-App` project specifically (`k logs deploy/coredesk-app` is
one of its own documented health checks) - this is the exact legacy
automation oss_infra_ops#156 already describes as having managed
oss02's old MicroK8s/CoreDesk setup.

Role list: `system_base`, `node_provision`, `k8s_prep`, `monitoring`,
`argocd`, `database_operator`, `app_deploy` - no overlap in naming or
structure with this repo's roles (`common`, `containerd`, `kubeadm`,
`control_plane`, `workers`, `operators`, `ingress`, `security`). It
also ships its own CoreDNS `Corefile` patch (`ansible/dns-fix.yaml`,
forwarding to `8.8.8.8`/`1.1.1.1`) that this repo has no equivalent of
and that was not evaluated for whether it's still needed anywhere.

## Why this repo (`k8s-dev-cluster`) is the one to keep

It already matches the live 4-node production cluster far more closely
(see `docs/software-baseline-audit.md`, oss_infra_ops#151) - real K8s/
Calico version alignment, a `site.yml` structured around the actual
topology, GitOps-compatible (ArgoCD watches `k8s-manifests`, not this
repo, for application state - this repo only ever provisions the hosts
themselves). `slu-cluster-mgmt` predates and assumes a cluster topology
(2 nodes, CoreDesk-specific) that no longer exists.

## What "consolidate" actually means here

Nothing from `slu-cluster-mgmt` needs porting into `k8s-dev-cluster` -
its roles solve problems for a topology and app (CoreDesk on a 2-node
cluster) that aren't this cluster's current reality, and its DNS fix
wasn't confirmed as still-needed anywhere live. "Consolidation" is
**deprecation**, not a merge of role content:

1. Rotate the exposed key (oss_infra_ops#165) - first, always.
2. Confirm with the CoreDesk/SLUCAM team (oss_infra_ops#156's own task)
   whether CoreDesk itself is still a live project needing onboarding
   onto `k8s-manifests` properly, or can be considered fully
   decommissioned - that answer determines whether anything from
   `slu-cluster-mgmt` is worth keeping at all, even as reference.
3. Add a deprecation notice to `slu-cluster-mgmt`'s own README pointing
   here, and/or archive the repository on GitHub (not done in this
   branch - a repo-settings change, not a file this branch can commit).
4. This repo (`k8s-dev-cluster`) becomes the sole source of truth for
   bare-metal host automation going forward - nothing here needs to
   change to make that true, it already is the complete, current
   automation for all 4 real nodes.

## Not done in this branch

- Rotating the key (oss_infra_ops#165 - a live credential change, not
  a documentation task).
- Archiving `slu-cluster-mgmt` on GitHub (repo settings, needs org
  permissions and a decision on sequencing with the key rotation
  above).
- The CoreDesk/SLUCAM stakeholder conversation (oss_infra_ops#156).
