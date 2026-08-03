---
tags:
  - service-mesh
  - must-gather
  - troubleshooting
---

# Collecting OpenShift Service Mesh Must-Gather

## Overview

The Service Mesh must-gather image collects diagnostic information specific to OpenShift Service Mesh (OSSM), including the control plane, Envoy sidecars, Istio resources, and related configuration.

## Command

```bash
oc adm must-gather \
  --image=registry.redhat.io/openshift-service-mesh/istio-must-gather-rhel9:3.4
```

### Notes

```bash
$ ls -1
event-filter.html
inspect.local.<hash>
must-gather.logs
registry-redhat-io-openshift-service-mesh-istio-must-gather-rhel9-sha256-<hash>
timestamp

$ tree . -L 2
. 
├── event-filter.html
├── inspect.local.<hash> 
│   ├── aggregated-discovery-apis.yaml
│   ├── aggregated-discovery-api.yaml
│   ├── cluster-scoped-resources 
│   ├── namespaces 
│   └── timestamp
├── must-gather.logs
├── registry-redhat-io-openshift-service-mesh-istio-must-gather-rhel9-sha256-<hash> 
│   ├── aggregated-discovery-apis.yaml
│   ├── aggregated-discovery-api.yaml
│   ├── cluster-scoped-resources 
│   ├── event-filter.html
│   ├── gather.logs
│   ├── namespaces 
│   ├── timestamp
│   └── version
└── timestamp

; first thing , look into must-gather.logs
Using must-gather plug-in image: registry.redhat.io/openshift-service-mesh/istio-must-gather-rhel9:3.4.0

ClusterID: <clusterid>
ClientVersion: 4.18.0
ClusterVersion: Stable at <clusterversion>
ClusterOperators:
        All healthy and stable

; cluster seems to be healthy.

$ omc use registry-redhat-io-openshift-service-mesh-istio-must-gather-rhel9-sha256-<hash>/

$ omc get istio

$ omc get gw ingressgateway-gateway -A

$ omc get virtualservice -A 

$ oc get virtualservce example -o yaml | yq .spec
gateways:
  - istio-gateway-example/ingressgateway-gateway
hosts:
  - app.apps.example.com
http:
  - match:
      - uri:
          prefix: /
    route:
      - destination:
          host: host.svc.cluster.local
          port:
            number: 8080

;control plane scope check

$ oc get istio istio-instance -o yaml | yq .spec.values.meshConfig.discoverySelectors
- matchLabels:
    istio-discovery: istio-label
    
$ oc get namespace -l istio-discovery=istio-label
NAME                           STATUS   AGE
app1-example-com               Active   298d
app2-example-com               Active   290d

;For Istio to discover a namespace and bring it under the control plane's scope, the namespace must be labeled with the appropriate discovery selector. Without this label, the namespace is not included within the control plane's discovery scope.
```