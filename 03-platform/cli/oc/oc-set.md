---
tags:
  - cli
  - oc
---
# oc set

Configure application resources.

## Available Commands

- build-hook        Update a build hook on a build config
- build-secret      Update a build secret on a build config
- data              Update the data within a config map or secret
- deployment-hook   Update a deployment hook on a deployment config
- env               Update environment variables on a pod template
- image             Update the image of a pod template
- image-lookup      Change how images are resolved when deploying applications
- probe             Update a probe on a pod template
- resources         Update resource requests/limits on objects with pod templates
- route-backends    Update the backends for a route
- selector          Set the selector on a resource
- serviceaccount    Update the service account of a resource
- subject           Update the user, group, or service account in a role binding or cluster role binding
- triggers          Update the triggers on one or more objects
- volumes           Update volumes on a pod template

```
; The following command updates the htpasswd-secret secret in the openshift-config namespace 
; by using the content of the /tmp/htpasswd file.

$ oc set data secret/htpasswd-secret \
  --from-file htpasswd=/tmp/htpasswd -n openshift-config

; The following command lets you review the volumes that are mounted inside the pod
$ oc set volumes pod todo-https-548d64494c-sgk4p
 todo-https-548d64494c-sgk4p
  secret/todo-certs as tls-certs
    mounted at /usr/local/etc/ssl/certs
  unknown as kube-api-access-mb8qh
    mounted at /var/run/secrets/kubernetes.io/serviceaccount

; The following command lets you review the volumes that are mounted by the deployment
$ oc set volumes deployment todo-https
 todo-https
  secret/todo-certs as tls-certs
    mounted at /usr/local/etc/ssl/certs

; The following command adds the TEAM=red environment variable to the preceding deployment
$ oc set env deployment/my-app TEAM=red
deployment.apps/my-app updated
```


## Usage examples

### Patching image in a running POD

Locating the container name is key.

```bash
%> oc get pod/broken -o json | jq -r .spec.containers[].name
httpd

%> oc set image pod/broken \
   httpd=image-registry.openshift-image-registry.svc:5000/openshift/httpd
pod/broken image updated
```

