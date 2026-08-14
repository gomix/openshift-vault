---
tags:
  - lifecycle
---

# InstallPlan

An **InstallPlan** is an Operator Lifecycle Manager (OLM) resource that describes the set of actions required to install or upgrade an Operator.

InstallPlans are typically created automatically from a `Subscription` when OLM detects a new Operator version or a change that requires installation.

## Approval modes

A `Subscription` can use two approval strategies:

- `Automatic` — OLM automatically approves and executes the InstallPlan.
- `Manual` — the InstallPlan remains pending until explicitly approved
    

## Inspecting InstallPlans

```bash
oc get installplan -n <namespace>
```

```bash
oc describe installplan <installplan-name> -n <namespace>
```

## Approving an InstallPlan

For subscriptions configured with manual approval:

```bash
oc patch installplan <installplan-name> \
  -n <namespace> \
  --type merge \
  -p '{"spec":{"approved":true}}'
```

## Related resources

```text
Subscription
    ↓
InstallPlan
    ↓
ClusterServiceVersion (CSV)
    ↓
Operator
```