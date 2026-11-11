# Software baseline audit vs. live production (oss_infra_ops#151)

Comparison of what this repo's Ansible roles would install on a fresh
node against what the real 4-node cluster is actually running today
(checked live via `kubectl`, `node-diag`'s read-only endpoints, and this
repo's own task files - no SSH access to any node was available this
session).

## Confirmed matching - no change needed

- **Kubernetes**: roles pin the apt repo to `pkgs.k8s.io/core:/stable:/v1.32`
  (`roles/kubeadm/tasks/main.yml`). Live: all 4 nodes report
  `v1.32.13` (`kubectl get nodes -o wide`). The `:/stable:/v1.32`
  channel tracks all 1.32.x patches automatically - this is correctly
  aligned, not stale.
- **Calico version**: pinned to `v3.27.0` in the security role. Live:
  `calico/node:v3.27.0` and `calico/cni:v3.27.0` images confirmed via
  `crictl images` on oss02 (through `node-diag`). Matches exactly.

## Found and fixed in this branch - a real design mismatch, not a version drift

The bigger problem wasn't a stale version number - it was that this
repo's roles describe a **different CNI architecture** than what's
actually running:

- `control_plane`'s role used to install **Flannel** as the primary
  CNI, with `security`'s role layering Calico on top in
  **policy-only** mode (NetworkPolicy enforcement only, not actual pod
  networking/IPAM) - this is a legitimate, real pattern, just not the
  one this cluster actually uses anymore.
- Live, confirmed directly: **zero Flannel pods or DaemonSets exist
  anywhere** in the cluster. `calico-node` is a real DaemonSet (4/4
  nodes ready), with a genuine `default-ipv4-ippool` IPPool CRD -
  Calico is the actual CNI + IPAM provider, not just a policy layer.

This matches a historical incident mentioned elsewhere in this
cluster's documentation (`oss_infra_ops`'s offline-infra-copilot
runbook references an "oss04 Flannel-leftover NetworkPolicy fix") -
the cluster evidently migrated off Flannel onto full Calico at some
point, and this repo's automation was never updated to reflect it. A
fresh node provisioned with the old roles would have ended up on a
genuinely different (and no-longer-intended) networking stack than
the rest of the live cluster.

**Fixed**: removed the Flannel install task from `control_plane`;
`security`'s Calico task now installs the full `calico.yaml` manifest
(CNI + IPAM + policy), patched to use this cluster's real pod CIDR
(`10.244.0.0/16`, same value `kubeadm init --pod-network-cidr` already
sets) instead of upstream's own default.

## Not independently verified this session (flagged, not guessed at)

- **containerd version** (`roles/containerd/tasks/main.yml` builds
  from a pinned upstream tarball URL - didn't cross-check the exact
  version string against a live node's `containerd --version`; no SSH
  access to run that command).
- **CNI plugins** (`containernetworking/plugins` pinned to `v1.4.0`) -
  not independently verified against what's actually on a live node's
  disk at `/opt/cni/bin`.

## Not tested end-to-end - this whole branch needs a real dry run before trusting it

**None of these Ansible changes have been run as part of a real
`ansible-playbook` execution.** This repo provisions bare-metal nodes
from scratch - there's no staging VM or spare node in this session to
execute `site.yml` against, and intentionally not attempted against any
of the 4 live production nodes (that would be a from-scratch reprovision
of a node currently serving real traffic).

**What was verified, directly**: the `replace` task patching Calico's
`CALICO_IPV4POOL_CIDR` env var was tested against a real, freshly
downloaded copy of `calico.yaml` v3.27.0 (the actual file this task
fetches) with the exact same regex Ansible's `replace` module would run
(Python's `re.sub`, `MULTILINE` - confirmed that's `replace`'s default
mode). It matches the real file's exact structure
(`            # - name: CALICO_IPV4POOL_CIDR` /
`            #   value: "192.168.0.0/16"`, 12-space indent) and produces
the expected output. This specific step is confirmed correct, just not
run through Ansible itself.

**What's still unverified**: the full play, end to end, against a real
node - task ordering across roles, `become`/`environment` correctness,
and whether anything else assumes Flannel was installed earlier in the
run. Dry-run (`ansible-playbook --check` won't actually validate `shell`
tasks meaningfully) or a real throwaway-node run is the only way to
close that out.

## Recommendation

The actual content fix (removing Flannel, installing full Calico with
the right CIDR) is correct and its one tricky step is verified against
the real upstream file. What's missing is a real end-to-end run,
which needs hardware this session doesn't have - merge once that's
done, or at least once someone's confirmed nothing else in the role
ordering assumed Flannel was present.
