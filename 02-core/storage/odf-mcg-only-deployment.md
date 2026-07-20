
# OpenShift Data Foundation (ODF) – Multi-Cloud Object Gateway (MCG) Only Deployment

## Definition

An MCG-only deployment installs only the NooBaa Multi-Cloud Object Gateway component of OpenShift Data Foundation, without deploying the Ceph storage cluster. 

In this architecture, NooBaa provides an S3-compatible object storage endpoint while using existing object storage backends (such as AWS S3, Azure Blob Storage, or external Ceph RGW) instead of local ODF-managed storage.

## Typical Use Cases

- Provide an S3-compatible endpoint for applications (e.g., Quay) without deploying a full Ceph storage cluster.
- Aggregate and manage multiple object storage backends through a single S3 endpoint.
- Enable hybrid and multi-cloud object storage with bucket replication, namespace buckets, and data tiering.
- Reduce infrastructure footprint when block (RBD) and file (CephFS) storage are not required.
- Integrate with existing enterprise object storage while benefiting from NooBaa's management and policy capabilities.

> In summary: An MCG-only deployment is the preferred option when only S3 object storage services are needed, eliminating the operational overhead of deploying and managing a full Ceph-based ODF cluster.

```
                   +----------------------+
                   |   Client Application |
                   |   (e.g. Quay, Apps)  |
                   +----------+-----------+
                              |
                              | S3 API
                              |
                   +----------v-----------+
                   |      NooBaa MCG      |
                   |  (S3 Gateway/Control)|
                   +----------+-----------+
                              |
              +---------------+----------------+
              |                                |
      +-------v--------+               +-------v--------+
      | AWS S3 Bucket  |               | External S3    |
      |                |               | (Ceph RGW,     |
      |                |               | Azure, etc.)   |
      +----------------+               +----------------+
```