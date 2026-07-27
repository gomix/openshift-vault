## Installing Quay on OpenShift Local

```
$ oc get packagemanifests | grep -i quay
quay-bridge-operator                                    Red Hat Operators     435d
project-quay                                            Community Operators   435d
quay-operator                                           Red Hat Operators     435d

$ oc new-project quay-enterprise

$ oc get packagemanifest quay-operator -o json |
jq -r '.status.channels[] | [.name, .currentCSV] | @tsv'
quay-v3.4       quay-operator.v3.4.7
quay-v3.5       quay-operator.v3.5.7
stable-3.10     quay-operator.v3.10.23
stable-3.11     quay-operator.v3.11.13
stable-3.12     quay-operator.v3.12.19
stable-3.13     quay-operator.v3.13.11
stable-3.14     quay-operator.v3.14.8
stable-3.15     quay-operator.v3.15.5
stable-3.16     quay-operator.v3.16.4
stable-3.17     quay-operator.v3.17.3
stable-3.6      quay-operator.v3.6.10
stable-3.7      quay-operator.v3.7.14
stable-3.8      quay-operator.v3.8.15
stable-3.9      quay-operator.v3.9.23

$ cat quay-operator-3-14-subscription.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: quay-operator
  namespace: openshift-operators
spec:
  channel: stable-3.14
  installPlanApproval: Automatic
  name: quay-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace

$ oc apply -f quay-operator-3-14-subscription.yaml 
subscription.operators.coreos.com/quay-operator created

$ oc get subscription quay-operator -n openshift-operators
NAME            PACKAGE         SOURCE             CHANNEL
quay-operator   quay-operator   redhat-operators   stable-3.14

$ oc get csv -n openshift-operators | grep quay
quay-operator.v3.14.8   Red Hat Quay   3.14.8    quay-operator.v3.14.7   Succeeded

$ oc get deployment -n openshift-operators -n openshift-operators -l operators.coreos.com/quay-operator.openshift-operators
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
quay-operator.v3.14.8   1/1     1            1           4m42s

%> oc get pods -n openshift-operators | grep quay
quay-operator.v3.14.8-548b9ddc98-nf8ff   1/1     Running   0          7m14s
```
