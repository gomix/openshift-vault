---
tags:
  - pods
  - lifecycle
---

A Pod may occasionally remain stuck in the `Terminating` state longer than expected. 

This usually happens when Kubernetes cannot complete the shutdown process due to finalizers, unavailable nodes, storage issues, or containers that do not terminate gracefully.

# Example 1 : my new cluster was sleeping

- My new lab  cluster was sleeping, meaning they nodes were turned off yesterday to save money and now they are back on.

## Gathering evidence

```
%> oc get pod
NAME                           READY   STATUS        RESTARTS   AGE
oras-login-test-sn5g8          0/1     Terminating   0          17h

%> oc get pod oras-login-test-sn5g8
NAME                    READY   STATUS        RESTARTS   AGE
oras-login-test-sn5g8   0/1     Terminating   0          17h
%> oc get pod oras-login-test-sn5g8 -o wide
NAME                    READY   STATUS        RESTARTS   AGE   IP             NODE                                           NOMINATED NODE   READINESS GATES
oras-login-test-sn5g8   0/1     Terminating   0          17h   10.130.0.169   ip-10-0-80-255.eu-central-1.compute.internal   <none>           <none>

; check for finalizers
%> oc get pod oras-login-test-sn5g8 -o json | jq '.metadata.finalizers'
null

; look into deletionTimestamp
%> oc get pod oras-login-test-sn5g8 -o json | jq '{
  deletionTimestamp: .metadata.deletionTimestamp,
  finalizers: .metadata.finalizers,
  node: .spec.nodeName
}'
{
  "deletionTimestamp": "2026-08-14T08:35:57Z",
  "finalizers": null,
  "node": "ip-10-0-80-255.eu-central-1.compute.internal"
}

%> date -u
Fri Aug 14 08:56:29 AM UTC 2026
```

## Analysis

Data:
- `deletionTimestamp`: `08:35:57 UTC`
	- Kubernetes API Server accepted  the deletion.
- Current time : `08:56:29 UTC`
- `finalizers: null`
	- No  finalizer holding the pod from termination.
- node: `ip-10-0-80-255.eu-central-1.compute.internal`

 > [!NOTE] 
 > - There is something happening with the Kubernetes API.
 > - Nodes were Ready.
 > - Checked CSRs and found the culprit.

```
%> oc get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                                                                   REQUESTEDDURATION   CONDITION
csr-5jr2k   74s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-bnld4   16m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-c8ndf   16m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-jfxvt   75s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-kpz9s   32m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-t2m48   16m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-vnr2r   32m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-x5pj2   32m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
csr-zvn5t   74s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending

; CSR related to the node-bootstrapping procedure
; Inspect subject of the CSR to confirm

%> for csr in $(oc get csr -o name); do                                          
    printf "%-20s " "$csr"                   
    oc get "$csr" -o jsonpath='{.spec.request}' \                                                 
      | base64 -d \                                 
      | openssl req -noout -subject                                                       
done                                                                                                                                                                              
certificatesigningrequest.certificates.k8s.io/csr-5jr2k subject=O=system:nodes, CN=system:node:ip-10-0-12-232.eu-central-1.compute.internal                                       
certificatesigningrequest.certificates.k8s.io/csr-bnld4 subject=O=system:nodes, CN=system:node:ip-10-0-12-232.eu-central-1.compute.internal                                       
certificatesigningrequest.certificates.k8s.io/csr-c8ndf subject=O=system:nodes, CN=system:node:ip-10-0-55-87.eu-central-1.compute.internal                                        
certificatesigningrequest.certificates.k8s.io/csr-jfxvt subject=O=system:nodes, CN=system:node:ip-10-0-55-87.eu-central-1.compute.internal                                        
certificatesigningrequest.certificates.k8s.io/csr-kpz9s subject=O=system:nodes, CN=system:node:ip-10-0-55-87.eu-central-1.compute.internal                                        
certificatesigningrequest.certificates.k8s.io/csr-t2m48 subject=O=system:nodes, CN=system:node:ip-10-0-80-255.eu-central-1.compute.internal                                       
certificatesigningrequest.certificates.k8s.io/csr-vnr2r subject=O=system:nodes, CN=system:node:ip-10-0-12-232.eu-central-1.compute.internal                                       
certificatesigningrequest.certificates.k8s.io/csr-x5pj2 subject=O=system:nodes, CN=system:node:ip-10-0-80-255.eu-central-1.compute.internal                                       
certificatesigningrequest.certificates.k8s.io/csr-zvn5t subject=O=system:nodes, CN=system:node:ip-10-0-80-255.eu-central-1.compute.internal  

; Approve and wait for cluster to reconcile, might take few minutes.
; Follow progress with oc get co
; No node reboot observed
```

## Root cause

* This is a new cluster a provisioned yesterday.
* CSRs from original ignition files expired after 24 hours.
* Cluster shut down before renewals.

## References
* [platform-agnostic](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/installing_on_any_platform/index?#installation-user-infra-generate-k8s-manifest-ignition_installing-platform-agnostic*\)




