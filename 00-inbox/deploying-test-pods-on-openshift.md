Mission deploy test pods to test with.

## **Imperative way examples**

```
# fedora minimal rawhide 
%> oc run testpod -it --rm --image registry.fedoraproject.org/fedora-minimal:rawhide --command -- /bin/bash  

# rhel7  
%> oc run testpod -it --rm --image registry.redhat.io/rhel7/rhel-tools:latest --command -- /bin/bash   

# rhel8 s2i-core 
%> oc run testpod --rm -it --image registry.redhat.io/rhel8/s2i-core --command -- /bin/bash       

# buildah 
%> oc run testpod --rm -it --image quay.io/buildah/stable --command -- /bin/bash  

# podman: using it for testing login to image registry 
%> oc run testpod --rm -it --image quay.io/podman/stable --command -- /usr/bin/podman login -u ggomezsa -p sha256~secret-token image-registry.openshift-image-registry.svc:5000 --tls-verify=false
If you don't see a command prompt, try pressing enter.                                                                                       Error attaching, falling back to logs: unable to upgrade connection: container ggomezsa-test not found in pod ggomezsa-test_openshift-image-registry
Login Succeeded!    

# aarch64 
%> oc run ggomezsa-test -it --rm --image registry.fedoraproject.org/fedora-minimal:37-aarch64 --command -- /bin/bash
```


##### **Declarative way**

`pod.yaml`

```
apiVersion: "v1"
kind: "Pod"
metadata:
  name: "testpod"
  labels:
    name: "infinidat-testpod"
spec:
  containers:
  - command:
    - /bin/bash
    name: "test-container"
    image: "registry.redhat.io/rhel7/rhel-tools:latest"
    tty: true
    volumeMounts:
    - mountPath: "/mnt/pvol"
      name: "pvol"
  volumes:
  - name: "pvol"
    persistentVolumeClaim:
      claimName: "testclaim1"
```

## Troubleshooting

### Quotas can get on your way
```
; from oc get event
Warning  FailedCreate  replicaset/mi-db-69d6b5684c                  
Error creating: pods "mi-db-69d6b5684c-f79tk" is forbidden: 
failed quota: entrenamiento-db-quota: must specify requests.cpu for: mysql; requests.memory for: mysql

; quota is enforcing you to specify requests in this case
; but you can not use --limits or --request anymore , then imperative way does not work anymore 
; switching to declarative way
%> cat pod.yaml 
apiVersion: "v1"
kind: "Pod"
metadata:
  name: "testpod"
spec:
  containers:
  - command:
    - /bin/bash
    name: "test-container"
    image: "registry.redhat.io/rhel7/rhel-tools:latest"
    tty: true
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
        
%> oc apply -f pod.yaml 
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "test-container" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "test-container" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "test-container" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "test-container" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
pod/testpod created

%> oc get pod
NAME      READY   STATUS    RESTARTS   AGE
testpod   1/1     Running   0          43s
```

### Fixing SCC warning

```
; from
%> oc apply -f pod.yaml 
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "test-container" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "test-container" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "test-container" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "test-container" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
...

; fix specifying SCC for the Pod and the containers
%> cat pod.yaml 
apiVersion: "v1"
kind: "Pod"
metadata:
  name: "testpod"
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
  - command:
    - /bin/bash
    name: "test-container"
    image: "registry.redhat.io/rhel7/rhel-tools:latest"
    tty: true
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
          
%> oc apply -f pod.yaml 
pod/testpod created
```