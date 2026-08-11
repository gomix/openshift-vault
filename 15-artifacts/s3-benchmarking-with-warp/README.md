---
tags:
  - openshift
  - object-storage
  - s3
  - benchmarking
  - warp
---
# S3 Benchmarking with Warp

This directory contains the OpenShift manifests and supporting artifacts used to benchmark S3-compatible object storage using MinIO Warp.

The benchmark suite was initially validated against NooBaa and includes workloads designed to approximate some of the I/O characteristics of a container registry such as Quay.

## Contents

- Benchmark namespace and ObjectBucketClaim (OBC)
- Warp client and service
- Persistent storage for benchmark results
- Warp analyzer
- Benchmark Jobs covering:
  - PUT and GET object-size sweeps
  - Concurrency sweeps
  - Metadata operations
  - Multipart uploads
  - Mixed workloads
  - Stress tests
  - Soak tests

## Documentation

See the accompanying technical note for the benchmark workflow, workload profiles, and result analysis.