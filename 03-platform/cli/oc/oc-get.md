---
tags:
  - cli
  - oc
  - resources
---

# oc get

Display one or many resources.

## Usage examples

```
$ oc get deployment -l app=test
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
test   1/1     1            1           3m43s

$ oc get operatorconditions
NAME                                    AGE
mcg-operator.v4.14.6-rhodf              3h25m
ocs-operator.v4.14.6-rhodf              3h25m
odf-csi-addons-operator.v4.14.6-rhodf   3h24m
odf-operator.v4.14.6-rhodf              3h42m

%> oc get infrastructure cluster -o json | jq .spec
{
  "cloudConfig": {
    "key": "config",
    "name": "cloud-provider-config"
  },
  "platformSpec": {
    "aws": {},
    "type": "AWS"
  }
}

%> oc get infrastructure cluster -o json | jq .status
{
  "apiServerInternalURI": "https://api-int.gizmo.sandbox730.opentlc.com:6443",
  "apiServerURL": "https://api.gizmo.sandbox730.opentlc.com:6443",
  "controlPlaneTopology": "HighlyAvailable",
  "cpuPartitioning": "None",
  "etcdDiscoveryDomain": "",
  "infrastructureName": "gizmo-46w5t",
  "infrastructureTopology": "HighlyAvailable",
  "platform": "AWS",
  "platformStatus": {
    "aws": {
      "region": "eu-central-1"
    },
    "type": "AWS"
  }
}

%> oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.14.18   True        False         5h43m   Cluster version is 4.14.18
```

## References

- [Getting Started with OpenShift CLI (v4.22)](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/cli_tools/openshift-cli-oc)