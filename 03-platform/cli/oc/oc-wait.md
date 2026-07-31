---
tags:
  - cli
  - oc
  - resources
  - automation
---
# oc wait

Wait for a specific condition on one or many resources.

#### Wait for all `ClusterOperator` to be available Available.

```
%> oc wait co --all --for=condition=Available=True --timeout=10m
clusteroperator.config.openshift.io/authentication condition met
clusteroperator.config.openshift.io/baremetal condition met
clusteroperator.config.openshift.io/cloud-controller-manager condition met
clusteroperator.config.openshift.io/cloud-credential condition met
clusteroperator.config.openshift.io/cluster-autoscaler condition met
clusteroperator.config.openshift.io/config-operator condition met
clusteroperator.config.openshift.io/console condition met
clusteroperator.config.openshift.io/control-plane-machine-set condition met
clusteroperator.config.openshift.io/csi-snapshot-controller condition met
clusteroperator.config.openshift.io/dns condition met
clusteroperator.config.openshift.io/etcd condition met
clusteroperator.config.openshift.io/image-registry condition met
clusteroperator.config.openshift.io/ingress condition met
clusteroperator.config.openshift.io/insights condition met
clusteroperator.config.openshift.io/kube-apiserver condition met
clusteroperator.config.openshift.io/kube-controller-manager condition met
clusteroperator.config.openshift.io/kube-scheduler condition met
clusteroperator.config.openshift.io/kube-storage-version-migrator condition met
clusteroperator.config.openshift.io/machine-api condition met
clusteroperator.config.openshift.io/machine-approver condition met
clusteroperator.config.openshift.io/machine-config condition met
clusteroperator.config.openshift.io/marketplace condition met
clusteroperator.config.openshift.io/monitoring condition met
clusteroperator.config.openshift.io/network condition met
clusteroperator.config.openshift.io/node-tuning condition met
clusteroperator.config.openshift.io/olm condition met
clusteroperator.config.openshift.io/openshift-apiserver condition met
clusteroperator.config.openshift.io/openshift-controller-manager condition met
clusteroperator.config.openshift.io/openshift-samples condition met
clusteroperator.config.openshift.io/operator-lifecycle-manager condition met
clusteroperator.config.openshift.io/operator-lifecycle-manager-catalog condition met
clusteroperator.config.openshift.io/operator-lifecycle-manager-packageserver condition met
clusteroperator.config.openshift.io/service-ca condition met
clusteroperator.config.openshift.io/storage condition met
```