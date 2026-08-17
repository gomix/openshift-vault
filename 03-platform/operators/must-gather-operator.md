---
tags:
  - openshift
  - operator
  - must-gather
  - diagnostics
  - troubleshooting
  - support
---

# Must-Gather Operator

The **must-gather operator** automates the collection of diagnostic information from an OpenShift cluster. Instead of running `oc adm must-gather` manually, a cluster administrator creates a `MustGather` custom resource that defines how the diagnostic data should be collected and uploaded to a Red Hat support case.

The operator can collect the standard OpenShift must-gather data, optionally include audit logs, and automatically clean up completed `MustGather` resources and their associated jobs.

## References

- [must-gather-operator source code](https://github.com/openshift/must-gather-operator)