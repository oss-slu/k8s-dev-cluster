# Branch status: burst-issue#152-deprecate-slu-cluster-mgmt

**Addresses:** oss_infra_ops#152 (Consolidate Host Automation and Deprecate Legacy slu-cluster-mgmt)
**Status: documentation ready to merge. Found something more urgent along the way - read that first.**

## Read first: oss_infra_ops#165

Cloning `slu-cluster-mgmt` to assess it for this issue turned up a
**committed root SSH private key** for `oss01`, live for ~8 months.
Filed as its own issue since it's a live credential exposure, not
something that should wait on this consolidation's timeline. Not
rotated, removed, or otherwise acted on here - that's a real sequencing
decision (rotate before touching the repo further) for whoever owns
this.

## What's here

`docs/legacy-slu-cluster-mgmt-deprecation.md` - actually cloned and
read `slu-cluster-mgmt` (not available in this sandbox otherwise) to
answer what it does, concluded "consolidate" means **deprecate**, not
merge: it's built for a 2-node topology and the `CoreDesk-App` project
specifically, with an entirely different, non-overlapping role
structure than this repo - nothing in it needs porting into
`k8s-dev-cluster`, which already fully covers the real 4-node cluster
(see the companion `burst-issue#151-...` branch's software-baseline
audit).

## What's not done, and why

- Rotating the exposed key and archiving the old repo - real actions
  with sequencing dependencies on each other, not something to do
  unattended.
- The actual CoreDesk/SLUCAM stakeholder conversation this issue (and
  #156) asks for - a conversation, not a technical task.

## Recommendation

Merge the documentation now. Treat oss_infra_ops#165 as the actually
urgent item this investigation surfaced.
# Branch status: burst-issue#151-modernize-ansible-roles

**Addresses:** oss_infra_ops#151 (Modernize Ansible Roles to Match Current Software Baseline)
**Status: content fix is correct and its trickiest step is verified against a real file; WIP until someone can run it end to end against real or throwaway hardware.**

## What's here

`docs/software-baseline-audit.md` - full comparison of what this repo's
roles would install vs. what the live cluster actually runs:

- **Kubernetes and Calico version pins already matched live versions** -
  confirmed, not stale, nothing to fix there (the issue's framing
  assumed more drift than was actually present).
- **The real finding**: `control_plane`'s role installed **Flannel** as
  the primary CNI, with `security`'s role layering Calico in
  **policy-only** mode on top - a legitimate pattern, just not what
  this cluster actually runs. Confirmed live: zero Flannel pods/
  DaemonSets anywhere; `calico-node` is a real 4/4 DaemonSet with a
  genuine IPPool CRD, i.e. Calico is the real CNI + IPAM provider. This
  connects to a historical incident already documented elsewhere in
  this cluster's docs ("oss04 Flannel-leftover NetworkPolicy fix") -
  the cluster migrated off Flannel at some point and this repo's
  automation was never updated to match.

**Fixed**: removed the Flannel install task; `security`'s Calico task
now installs the full manifest (not policy-only), patched to this
cluster's real pod CIDR (`10.244.0.0/16`) instead of upstream's
default.

## What was actually verified (not just asserted)

The one genuinely tricky part - patching Calico's commented-out
`CALICO_IPV4POOL_CIDR` env var via Ansible's `replace` module - was
tested against a real, freshly downloaded copy of the exact file this
task fetches (`calico.yaml` v3.27.0), using the same regex semantics
Ansible's `replace` module actually uses (Python `re.sub` with
`MULTILINE`). It matches and produces the expected result. This isn't
guessed at.

## What's not verified

The full play, end to end, against a real node - no spare/throwaway
hardware exists in this session to run `ansible-playbook site.yml`
against, and it wasn't attempted against any of the 4 live production
nodes (that would mean reprovisioning a node serving real traffic from
scratch). See `docs/software-baseline-audit.md`'s own "Not tested"
section for exactly what that would need to confirm.

Also flagged, not independently checked: containerd version and CNI
plugin version pins (`roles/containerd/tasks/main.yml`) weren't
cross-referenced against a live node's actual installed versions - no
SSH access this session (same limitation as the companion
`oss_infra_ops` branch `burst-issue#156-oss02-legacy-audit`).

## Recommendation

The content fix is right and its riskiest step is confirmed correct
against the real file. Don't merge blind, though - get a real
`ansible-playbook` run against a throwaway node first, or at minimum
have someone familiar with this repo's role ordering confirm nothing
else implicitly depended on Flannel being installed earlier in the
play.
