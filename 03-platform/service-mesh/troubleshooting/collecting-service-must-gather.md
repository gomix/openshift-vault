
# Collecting OpenShift Service Mesh Must-Gather

## Overview

The Service Mesh must-gather image collects diagnostic information specific to OpenShift Service Mesh (OSSM), including the control plane, Envoy sidecars, Istio resources, and related configuration.

## Command

```bash
oc adm must-gather \
  --image=registry.redhat.io/openshift-service-mesh/istio-must-gather-rhel9:3.4
