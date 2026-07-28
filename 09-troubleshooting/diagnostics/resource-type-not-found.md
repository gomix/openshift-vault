---
tags:
  - oc
  - api
  - resource
---

# Resource Type Not Found

`oc` returns *"the server doesn't have a resource type"* when the requested API resource is not available in the connected cluster.

## Case 1: Route Resource

```
$ oc get routes
error: the server doesn't have a resource type "routes"

; not present in api-resources
$ oc api-resources | grep -i route
adminpolicybasedexternalroutes      apbexternalroute          k8s.ovn.org/v1                                false        AdminPolicyBasedExternalRoute
egressrouters                                                 network.operator.openshift.io/v1              true         EgressRouter

; not in api-versions
%> oc api-versions | grep route
%> <nothing>

; CRD not present
$ oc get crd | grep -i route
adminpolicybasedexternalroutes.k8s.ovn.org                        2026-07-27T06:05:02Z
egressrouters.network.operator.openshift.io                       2026-07-27T06:01:41Z

;co operators looks good
$ oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
authentication                             4.18.25   True        False         False      27h
baremetal                                  4.18.25   True        False         False      27h
cloud-controller-manager                   4.18.25   True        False         False      27h
cloud-credential                           4.18.25   True        False         False      27h
cluster-autoscaler                         4.18.25   True        False         False      27h
config-operator                            4.18.25   True        False         False      27h
console                                    4.18.25   True        False         False      27h
control-plane-machine-set                  4.18.25   True        False         False      27h
csi-snapshot-controller                    4.18.25   True        False         False      21h
dns                                        4.18.25   True        False         False      21h
etcd                                       4.18.25   True        False         False      27h
image-registry                             4.18.25   True        False         False      27h
ingress                                    4.18.25   True        False         False      21h
insights                                   4.18.25   True        False         False      27h
kube-apiserver                             4.18.25   True        False         False      27h
kube-controller-manager                    4.18.25   True        False         False      27h
kube-scheduler                             4.18.25   True        False         False      27h
kube-storage-version-migrator              4.18.25   True        False         False      21h
machine-api                                4.18.25   True        False         False      27h
machine-approver                           4.18.25   True        False         False      27h
machine-config                             4.18.25   True        False         False      27h
marketplace                                4.18.25   True        False         False      27h
monitoring                                 4.18.25   True        False         False      27h
network                                    4.18.25   True        False         False      27h
node-tuning                                4.18.25   True        False         False      27h
olm                                        4.18.25   True        False         False      21h
openshift-apiserver                        4.18.25   True        False         False      21h
openshift-controller-manager               4.18.25   True        False         False      21h
openshift-samples                          4.18.25   True        False         False      27h
operator-lifecycle-manager                 4.18.25   True        False         False      27h
operator-lifecycle-manager-catalog         4.18.25   True        False         False      27h
operator-lifecycle-manager-packageserver   4.18.25   True        False         False      21h
service-ca                                 4.18.25   True        False         False      27h
storage                                    4.18.25   True        False         False      27h

; query directly the API, apigroup exists
$ oc get --raw /apis/route.openshift.io | jq .
{
  "kind": "APIGroup",
  "apiVersion": "v1",
  "name": "route.openshift.io",
  "versions": [
    {
      "groupVersion": "route.openshift.io/v1",
      "version": "v1"
    }
  ],
  "preferredVersion": {
    "groupVersion": "route.openshift.io/v1",
    "version": "v1"
  }
}

; checking client/server versions
$ oc version
Client Version: 4.18.25
Kustomize Version: v5.4.2
Server Version: 4.18.25
Kubernetes Version: v1.31.13

; ask for the api group
$ oc api-resources --api-group=route.openshift.io
NAME   SHORTNAMES   APIVERSION   NAMESPACED   KIND

; openshift apiserver looks good too
$ oc get pods -n openshift-apiserver
NAME                        READY   STATUS    RESTARTS   AGE
apiserver-67c894d8f-hh889   2/2     Running   2          27h
apiserver-67c894d8f-ltfkz   2/2     Running   2          27h
apiserver-67c894d8f-z9dlb   2/2     Running   2          27h

; lets look into the logs of it
$ oc logs -n openshift-apiserver \
  -l apiserver=true \
  --since=30m | grep -Ei 'route|503|error|panic'
Defaulted container "openshift-apiserver" out of: openshift-apiserver, openshift-apiserver-check-endpoints, fix-audit-permissions (init)
Defaulted container "openshift-apiserver" out of: openshift-apiserver, openshift-apiserver-check-endpoints, fix-audit-permissions (init)
Defaulted container "openshift-apiserver" out of: openshift-apiserver, openshift-apiserver-check-endpoints, fix-audit-permissions (init)
Error from server: Get "https://10.0.23.105:10250/containerLogs/openshift-apiserver/apiserver-67c894d8f-hh889/\
                   openshift-apiserver?sinceSeconds=1800&tailLines=10": remote error: tls: internal error

; this error typically comes because pending csr, confirm
$ oc get csr                                                   
NAME        AGE     SIGNERNAME                                    REQUESTOR                                                                   REQUESTEDDURATION   CONDITION
csr-2x9gg   40m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-47vjs   117m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-4rcqr   9m10s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-59jbc   40m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-75kw5   86m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-7795b   71m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-7zxc8   86m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-bf45w   148m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-bzk7x   55m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-cjkh2   148m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-flnpg   9m10s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-gkwsb   55m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-grmjn   40m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-gxvrl   132m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-j947w   24m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-jwqvf   117m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-khkwn   9m11s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-m2jjz   117m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-m5gfv   86m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-nbrkn   101m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-njblk   132m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-nmbts   24m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-nszr7   101m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-pcsdc   24m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-pdbn9   71m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-qft9w   101m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-tvgfj   132m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-vn2lt   148m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-wfdwl   71m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-zb95m   55m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending

; approve them all (carefull, u might want to check them individually if in production)
$ oc get csr --no-headers \
| awk '$NF=="Pending"{print $1}' \
| xargs -r oc adm certificate approve
certificatesigningrequest.certificates.k8s.io/csr-2x9gg approved
certificatesigningrequest.certificates.k8s.io/csr-47vjs approved
certificatesigningrequest.certificates.k8s.io/csr-4rcqr approved
certificatesigningrequest.certificates.k8s.io/csr-59jbc approved
certificatesigningrequest.certificates.k8s.io/csr-75kw5 approved
certificatesigningrequest.certificates.k8s.io/csr-7795b approved
certificatesigningrequest.certificates.k8s.io/csr-7zxc8 approved
certificatesigningrequest.certificates.k8s.io/csr-bf45w approved
certificatesigningrequest.certificates.k8s.io/csr-bzk7x approved
certificatesigningrequest.certificates.k8s.io/csr-cjkh2 approved
certificatesigningrequest.certificates.k8s.io/csr-flnpg approved
certificatesigningrequest.certificates.k8s.io/csr-gkwsb approved
certificatesigningrequest.certificates.k8s.io/csr-grmjn approved
certificatesigningrequest.certificates.k8s.io/csr-gxvrl approved
certificatesigningrequest.certificates.k8s.io/csr-j947w approved
certificatesigningrequest.certificates.k8s.io/csr-jwqvf approved
certificatesigningrequest.certificates.k8s.io/csr-khkwn approved
certificatesigningrequest.certificates.k8s.io/csr-m2jjz approved
certificatesigningrequest.certificates.k8s.io/csr-m5gfv approved
certificatesigningrequest.certificates.k8s.io/csr-nbrkn approved
certificatesigningrequest.certificates.k8s.io/csr-njblk approved
certificatesigningrequest.certificates.k8s.io/csr-nmbts approved
certificatesigningrequest.certificates.k8s.io/csr-nszr7 approved
certificatesigningrequest.certificates.k8s.io/csr-pcsdc approved
certificatesigningrequest.certificates.k8s.io/csr-pdbn9 approved
certificatesigningrequest.certificates.k8s.io/csr-qft9w approved
certificatesigningrequest.certificates.k8s.io/csr-tvgfj approved
certificatesigningrequest.certificates.k8s.io/csr-vn2lt approved
certificatesigningrequest.certificates.k8s.io/csr-wfdwl approved
certificatesigningrequest.certificates.k8s.io/csr-zb95m approved

; cluster will reconcile itself
$ oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
authentication                             4.18.25   True        False         False      27h     
baremetal                                  4.18.25   True        False         False      28h     
cloud-controller-manager                   4.18.25   True        False         False      28h     
cloud-credential                           4.18.25   True        False         False      28h     
cluster-autoscaler                         4.18.25   True        False         False      28h     
config-operator                            4.18.25   True        False         False      28h     
console                                    4.18.25   True        False         False      27h     
control-plane-machine-set                  4.18.25   True        False         False      27h     
csi-snapshot-controller                    4.18.25   True        False         False      21h     
dns                                        4.18.25   True        False         False      21h     
etcd                                       4.18.25   True        False         False      27h     
image-registry                             4.18.25   True        False         False      27h     
ingress                                    4.18.25   True        False         False      21h     
insights                                   4.18.25   True        False         False      28h     
kube-apiserver                             4.18.25   True        False         False      27h     
kube-controller-manager                    4.18.25   True        False         False      27h     
kube-scheduler                             4.18.25   True        False         False      27h     
kube-storage-version-migrator              4.18.25   True        False         False      21h     
machine-api                                4.18.25   True        False         False      27h     
machine-approver                           4.18.25   True        False         False      28h     
machine-config                             4.18.25   True        False         False      28h     
marketplace                                4.18.25   True        False         False      28h     
monitoring                                 4.18.25   True        False         False      27h     
network                                    4.18.25   True        True          False      28h     DaemonSet "/openshift-multus/network-metrics-daemon" is not available (awaiting 1 nodes)...
node-tuning                                4.18.25   True        False         False      28h     
olm                                        4.18.25   True        False         False      21h     
openshift-apiserver                        4.18.25   True        False         False      21h     
openshift-controller-manager               4.18.25   True        False         False      21h     
openshift-samples                          4.18.25   True        False         False      27h     
operator-lifecycle-manager                 4.18.25   True        False         False      28h     
operator-lifecycle-manager-catalog         4.18.25   True        False         False      28h     
operator-lifecycle-manager-packageserver   4.18.25   True        False         False      21h     
service-ca                                 4.18.25   True        False         False      28h     
storage                                    4.18.25   True        False         False      28h 

$ oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
authentication                             4.18.25   True        False         False      0s      
baremetal                                  4.18.25   True        False         False      28h     
cloud-controller-manager                   4.18.25   True        False         False      28h     
cloud-credential                           4.18.25   True        False         False      28h     
cluster-autoscaler                         4.18.25   True        False         False      28h     
config-operator                            4.18.25   True        False         False      28h     
console                                    4.18.25   True        False         False      27h     
control-plane-machine-set                  4.18.25   True        False         False      28h     
csi-snapshot-controller                    4.18.25   True        False         False      21h     
dns                                        4.18.25   True        False         False      21h     
etcd                                       4.18.25   True        False         False      28h     
image-registry                             4.18.25   True        False         False      27h     
ingress                                    4.18.25   False       True          False      4s      The "default" ingress controller reports Available=False: IngressControllerUnavailable: One or more status conditions indicate unavailable: DeploymentAvailable=False (DeploymentUnavailable: The deployment has Available status condition set to False (reason: MinimumReplicasUnavailable) with message: Deployment does not have minimum availability.)
insights                                   4.18.25   True        False         False      28h     
kube-apiserver                             4.18.25   True        False         False      27h     
kube-controller-manager                    4.18.25   True        False         False      27h     
kube-scheduler                             4.18.25   True        False         False      27h     
kube-storage-version-migrator              4.18.25   True        False         False      21h     
machine-api                                4.18.25   True        False         False      27h     
machine-approver                           4.18.25   True        False         False      28h     
machine-config                             4.18.25   True        False         False      28h     
marketplace                                4.18.25   True        False         False      28h     
monitoring                                 4.18.25   True        False         False      27h     
network                                    4.18.25   True        False         False      28h     
node-tuning                                4.18.25   True        False         False      28h     
olm                                        4.18.25   True        False         False      21h     
openshift-apiserver                        4.18.25   False       False         False      4s      APIServicesAvailable: apiservices.apiregistration.k8s.io/v1.apps.openshift.io: not available: failing or missing response from https://10.129.0.85:8443/apis/apps.openshift.io/v1: bad status from https://10.129.0.85:8443/apis/apps.openshift.io/v1: 401...
openshift-controller-manager               4.18.25   True        True          False      21h     Progressing: deployment/controller-manager: observed generation is 7, desired generation is 8....
openshift-samples                          4.18.25   True        False         False      27h     
operator-lifecycle-manager                 4.18.25   True        False         False      28h     
operator-lifecycle-manager-catalog         4.18.25   True        False         False      28h     
operator-lifecycle-manager-packageserver   4.18.25   False       True          False      3s      ClusterServiceVersion openshift-operator-lifecycle-manager/packageserver observed in phase Failed with reason: ComponentUnhealthy, message: apiServices not installed
service-ca                                 4.18.25   True        False         False      28h     
storage                                    4.18.25   True        False         False      28h     

; when done, fixed 
$ oc get routes
NAME                HOST/PORT                                                            PATH   SERVICES        PORT    TERMINATION     WILDCARD
quay-quay           quay.apps.lab.sandbox502.opentlc.com                                        quay-quay-app   http    edge/Redirect   None
quay-quay-builder   quay-quay-builder-quay-enterprise.apps.lab.sandbox2478.opentlc.com          quay-quay-app   grpc    edge/Redirect   None
quay-test-route     quay-test-route-quay-enterprise.apps.lab.sandbox2478.opentlc.com            none            <all>                   None
```

### Conclusion

The error the server doesn't have a resource type "routes" was not caused by a missing Route API or an incorrect oc client version. The root cause was a large number of pending kubelet client certificate signing requests (CSRs), which prevented proper communication between the control plane and the kubelets. As a result, the Route API discovery endpoint (/apis/route.openshift.io/v1) returned 503 ServiceUnavailable, causing oc to believe that the Route resource was unavailable. Approving the pending CSRs restored normal API discovery, and oc get routes started working again.

### Lessons Learned

- Do not assume that `the server doesn't have a resource type` always means the resource is missing.
- Verify the API discovery endpoints (`/apis/<group>` and `/apis/<group>/<version>`).
- Check for pending CSRs when API discovery or kubelet-related operations (`oc logs`, `oc exec`, `oc cp`) fail unexpectedly.
- A healthy `ClusterOperator` status does not necessarily guarantee that all API endpoints are functioning correctly.