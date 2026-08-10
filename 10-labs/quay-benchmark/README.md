# Quay Benchmark

## Overview

This laboratory contains a reproducible benchmarking framework for evaluating the performance, scalability, and behavior of Red Hat Quay under different workloads and deployment configurations.

The primary objective is not only to execute performance tests, but also to build a reusable benchmark suite that can be executed across different OpenShift environments and storage backends, producing consistent and comparable results.

The benchmark framework is intended to answer questions such as:

- How many OCI artifacts can Quay process per second?
- How does latency change as concurrency increases?
- How does storage backend selection affect performance?
- What are the resource requirements under different workloads?
- How do different Quay configurations compare?

---

# Objectives

- Build a repeatable benchmarking methodology.
- Validate Quay functionality before performance testing.
- Measure throughput and latency under controlled workloads.
- Compare multiple deployment configurations.
- Collect platform and application metrics.
- Produce reproducible benchmark reports.

---

# Benchmark Workflow

The benchmark process is divided into several phases.

## 1. Functional Validation

Verify that the benchmark environment is working correctly before collecting performance data.

Validation includes:

- Authentication
- Push OCI artifacts
- Pull OCI artifacts
- Verify artifact integrity
- Repository creation
- Repository deletion (optional)

---

## 2. Baseline Performance

Execute a benchmark using a single client with minimal concurrency.

This phase establishes the theoretical performance of the environment without significant resource contention.

Metrics include:

- Average latency
- Throughput
- Requests per second
- Transfer rate

---

## 3. Scalability Testing

Increase client concurrency to evaluate how Quay scales under load.

Typical concurrency levels include:

- 1 client
- 5 clients
- 10 clients
- 20 clients
- 50 clients
- 100 clients

The objective is to identify the point at which latency increases or throughput plateaus.

---

## 4. Dataset Evaluation

Execute the same benchmark using artifacts of different sizes.

Example datasets:

| Profile | Artifact Size |
|----------|--------------:|
| Small | 1 MB |
| Medium | 10 MB |
| Large | 100 MB |
| XL | 500 MB |
| XXL | 2 GB |

---

## 5. Mixed Workloads

Real-world registries rarely execute a single operation.

Representative workloads may include:

- Pull-heavy
- Push-heavy
- Balanced Push/Pull
- Read-only
- Artifact upload
- Metadata operations

---

## 6. Platform Comparison

Execute identical benchmark scenarios against different Quay deployments.

Examples:

- Local filesystem
- NooBaa
- Ceph RGW
- AWS S3
- MinIO
- Different storage classes
- Different OpenShift cluster sizes

---

# Metrics

Performance data should be collected from multiple layers.

## Client

- Requests per second
- Throughput
- Average latency
- P95 latency
- P99 latency
- Error rate

## Quay

- CPU utilization
- Memory utilization
- HTTP request rate
- HTTP response codes
- Request duration

## OpenShift

- Node CPU
- Node Memory
- Network utilization
- Persistent volume latency
- Persistent volume throughput

## Storage Backend

- IOPS
- Read latency
- Write latency
- Bandwidth
- Queue depth

---

# Expected Repository Structure

```text
quay-benchmark/
├── README.md
├── oras-client.md
├── benchmark-scenarios.md
├── datasets.md
├── metrics.md
├── automation.md
├── scripts/
├── manifests/
├── datasets/
├── results/
└── reports/
```

---

# Design Principles

This benchmark suite is designed to be:

- Reproducible
- Automated
- Storage-agnostic
- Cloud-agnostic
- OpenShift-native
- Easy to extend
- Easy to compare across environments

---

# Future Improvements

Potential future enhancements include:

- Automated report generation
- Prometheus integration
- Grafana dashboards
- Multiple concurrent benchmark clients
- Long-duration endurance testing
- Failure injection
- Network latency simulation
- Storage backend comparison
- Quay version comparison
- OpenShift version comparison