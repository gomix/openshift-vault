## Considerations

- Wanted to use OpenShift Local (CRC), tried, feasible, but my devices did not have enough resources, switching to AWS using environments provided by Red Hat.
- We want to use Quay 3.14.5.
- We want to use NooBaa/Multicloud Object Gateway to partially replicate
  customer architecture.

## Lab Cluster  on AWS

- Will use IPI method to deploy the 4.18.25 cluster.
- Will deploy Compact setup with enough resources to hold the workload.
	- For Compact with ODF + Quay                                                                
	- Instance type: 3 × m6a.4xlarge                                                                                                                                                                                             
		- 16 vCPU                                                                                                                                                                                                     
		- 64 GiB RAM por nodo
		- Root: 200–250 GiB gp3
		- Optional* ODF: 1 × 500 GiB gp3 EBS volume por nodo      

## Lab architecture and details

```
                 +---------------------+
                 |    Quay Operator    |
                 +----------+----------+
                            |
                    Quay Registry
                            |
              +-------------+--------------+
              |                            |
         PostgreSQL                    Redis
              |
           Clair
              |
              |
        S3 Object Storage
              |
              v
        NooBaa S3 Endpoint
              |
      BackingStore (PVC)
              |
 crc-csi-hostpath-provisioner
```
