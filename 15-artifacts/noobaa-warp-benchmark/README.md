# NooBaa WARP Benchmark Suite

This directory contains the Kubernetes manifests used to benchmark the S3 performance of Red Hat OpenShift Data Foundation (ODF) Multi-Cloud Gateway (NooBaa) using MinIO WARP.

## Contents

The suite includes manifests for:

- Namespace and benchmark resources
- ObjectBucketClaim (OBC)
- ConfigMaps and Secrets
- WARP benchmark Jobs

The benchmark Jobs cover multiple scenarios, including:

- PUT
- GET
- MIXED
- Multipart uploads
- Stress tests
- Soak tests

## Usage

Apply the manifests individually according to the benchmarking scenario you want to execute.

See the accompanying runbook for the complete benchmarking procedure, environment preparation, and result analysis (WiP).