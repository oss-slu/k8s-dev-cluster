# Branch status: burst-issue#150-fix-inventory-topology

**Addresses:** oss_infra_ops#150 (Update Cluster Inventory and Topology to 4-Node Live State)
**Status: ready to merge.**

## What was actually wrong

`inventory.ini` already had exactly 4 nodes (1 control plane + 3
workers), matching the live cluster's real node count - this wasn't a
missing-node problem. The real bug: one worker was named wrong. The
host at `10.60.63.5` was listed as `oss03`, but `kubectl get nodes -o
wide` confirms the real node at that IP is named **`oss05`** - there is
no `oss03` anywhere in the live cluster. Same stale name was also in
`README.md`'s node list.

This isn't a cosmetic typo: Ansible's inventory hostnames are used
throughout this repo's roles (`hosts: workers`, per-host task targeting,
etc.) - running `site.yml` against a host literally named `oss03`
when no such host exists (it would try to resolve/SSH to a hostname
that was never the real one) is a real, latent failure waiting to
happen the next time someone runs this playbook expecting it to
reach all 4 real nodes.

## What I didn't touch, and why

The `ansible_user` values (`workernode1`/`workernode2`/`workernode3`)
weren't changed - I have no way to verify which username is actually
correct for SSH to each host from this session (no SSH access
confirmed - see the companion `oss_infra_ops` branch
`burst-issue#156-oss02-legacy-audit` for where this was tested and
denied). Worth a human double-checking these match real `~/.ssh/config`
or known-working credentials before the next real `ansible-playbook`
run, but not something to guess-and-fix blind.

## Related, not fixed here (separate issue's scope)

`README.md`'s "Stack" section still claims "Flannel CNI" - confirmed
live, this is also stale (zero Flannel pods/DaemonSets exist cluster-
wide; `calico-node` is the real CNI DaemonSet, 4/4 nodes, with a real
Calico IPPool CRD). Left for the companion `burst-issue#151-...` branch
since it's a software-baseline question, not a topology one.
