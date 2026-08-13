---
title: Migrating AWS Load Balancers from CLB to NLB in OpenShift 4.18
tags:
  - load-balancer
  - clb
  - nlb
  - ipi
  - quay
  - troubleshooting
  - ocp-4.18
---

# Migrating AWS Load Balancers from CLB to NLB in OpenShift 4.18

## Introduction

This note documents the migration from AWS **Classic Load Balancers (CLB)** to **Network Load Balancers (NLB)** in an OpenShift Container Platform 4.18.25 cluster deployed on AWS using IPI.

The objective is to describe the required configuration changes, migration procedure, validation steps, and rollback considerations required to replace the existing CLBs while maintaining connectivity to the OpenShift API and application ingress endpoints.

## Motivation

The main reason for this change is a recurring issue observed when transferring large objects through the existing AWS Classic Load Balancer (CLB) path.

The problem was initially identified while testing **NooBaa S3 uploads**, where transfers of approximately **600 MB or larger** consistently failed while smaller objects completed successfully.

The same behavior was later observed with **Quay** when NooBaa was configured as its S3 object storage backend. In this scenario, pushing sufficiently large container images caused the underlying object uploads to NooBaa to fail in a similar manner.

This correlation indicates that the issue is not specific to Quay itself, but rather to the network path used to reach the S3 service, with the AWS Classic Load Balancer being a likely limiting component.

The migration from **CLB to NLB** is therefore intended to:

- Remove the Classic Load Balancer from the affected data path.
- Provide a Layer 4 load-balancing path better suited for long-lived TCP connections and large data transfers.
- Validate whether the observed failures are directly associated with the CLB architecture.
- Confirm successful large-object uploads directly to NooBaa.
- Confirm successful large-image pushes to Quay when NooBaa is used as the S3 backend.
- Document a reproducible migration, validation, and rollback procedure.
- Provide a basis for automating the same change across similar AWS IPI OpenShift environments.

The migration will be considered successful only after reproducing the original test conditions and confirming that transfers above the previously observed ~600 MB failure threshold complete successfully.

## Procedure

### First the failed test.

```
%> oc logs jobs/oras-push-large
=== Creating payload ===
Payload: payload.bin
Size:    1073741824 bytes

=== Push ===
Target: quay.apps.lab.example.com/admin/benchmark-phase1:large

Preparing payload.bin
Exists    44136fa355b3 application/vnd.oci.empty.v1+json
Uploading 49bc20df15e4 payload.bin
Error: Put "https://quay.apps.lab.example.com/v2/admin/benchmark-phase1/blobs/uploads/33099fe8-7e35-463b-833c-cf318cbb7a4e?digest=sha256%3A49bc20df15e412a64472421e13fe86ff1c5165e18b2afccf160d4dc19fe68a14": write tcp 10.130.0.148:56302->18.196.132.253:443: write: connection reset by peer
RESULT=FAIL
ORAS_EXIT_CODE=1
```

### Patch ingresscontroller

In this lab, the default `IngressController` is using an AWS Classic Load Balancer (CLB) with no additional customizations.

The following patch changes the `endpointPublishingStrategy` to use an AWS Network Load Balancer (NLB). 

The Ingress Operator reconciles the configuration and provisions the new load balancer automatically.

```
oc -n openshift-ingress-operator patch ingresscontroller/default \
  --type=merge \
  -p '{
    "spec": {
      "endpointPublishingStrategy": {
        "type": "LoadBalancerService",
        "loadBalancer": {
          "scope": "External",
          "providerParameters": {
            "type": "AWS",
            "aws": {
              "type": "NLB"
            }
          }
        }
      }
    }
  }'
```

### Monitor progress and confirm

Monitor progress.

```
watch -n 2 '
oc get ingresscontroller default -n openshift-ingress-operator;
echo;
oc get svc router-default -n openshift-ingress;
echo;
oc get co ingress
'
```

Confirm LB type in AWS.

```
%> aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[].{Name:LoadBalancerName,Type:Type,DNS:DNSName,State:State.Code}' \
  --output table
-----------------------------------------------------------------------------------------------------------------------------------------------
|                                                            DescribeLoadBalancers                                                            |
+-----------------------------------------------------------------------------------+------------------------------------+---------+----------+
|                                        DNS                                        |               Name                 |  State  |  Type    |
+-----------------------------------------------------------------------------------+------------------------------------+---------+----------+
|  lab-8wxt8-ext-521c52169ef921b8.elb.eu-central-1.amazonaws.com                    |  lab-8wxt8-ext                     |  active |  network |
|  lab-8wxt8-int-533a8a5aa6e9abe5.elb.eu-central-1.amazonaws.com                    |  lab-8wxt8-int                     |  active |  network |
|  ac31d301c45564660b1144e3e6286d99-c81b4cd51e5c7767.elb.eu-central-1.amazonaws.com |  ac31d301c45564660b1144e3e6286d99  |  active |  network |
+-----------------------------------------------------------------------------------+------------------------------------+---------+----------+
```

### Run the test

It should pass.

```
%> oc logs jobs/oras-push-large
=== Creating payload ===
Payload: payload.bin
Size:    1073741824 bytes

=== Push ===
Target: quay.apps.lab.example.com/admin/benchmark-phase1:large

Preparing payload.bin
Uploading 49bc20df15e4 payload.bin
Exists    44136fa355b3 application/vnd.oci.empty.v1+json
Uploaded  49bc20df15e4 payload.bin
Uploading 40c5d8545465 application/vnd.oci.image.manifest.v1+json
Uploaded  40c5d8545465 application/vnd.oci.image.manifest.v1+json
Pushed [registry] quay.apps.lab.example.com/admin/benchmark-phase1:large
ArtifactType: application/vnd.unknown.artifact.v1
Digest: sha256:40c5d85454656bedc63e196c778450f80245da316158708f3216d140a305c060

=== Results ===
TEST=push-large
SIZE_BYTES=1073741824
DURATION_SECONDS=48.367
THROUGHPUT_MIB_S=21.17
DIGEST=sha256:40c5d85454656bedc63e196c778450f80245da316158708f3216d140a305c060
RESULT=PASS
```

> [!warning] 
> - Scope: AWS IPI clusters using the default LoadBalancerService IngressController.
> - This procedure has been tested on an AWS IPI cluster. UPI environments might require additional load balancer configuration depending on how ingress infrastructure is managed.