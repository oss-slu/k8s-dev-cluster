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
