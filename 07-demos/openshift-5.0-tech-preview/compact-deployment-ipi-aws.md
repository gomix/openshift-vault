---
tags:
  - openshift
  - demo
  - tech-preview
  - openshift-5
  - deployment
---

> [!NOTE]
> This page documents test deployment of OpenShift Container Platform 5.0.0 Tech Preview. 
> The goal is to explore the new major release, become familiar with the environment, and capture early observations during deployment and validation.
> Diclaimer: this is not official Red Hat documentation.

## Downloadables

Early Engineering Candidates of OpenShift 5.0 are available at:
* https://console.redhat.com/openshift/install/multi/pre-release
* No changes, select and download installer and cli tools.

### Versions

```
%> openshift-install version
openshift-install 5.0.0-ec.5
built from commit d8f6a96a1c18ee540099ff73789fde54cc9b12cd
release image quay.io/openshift-release-dev/ocp-release@sha256:d1c0459e0a513c95a3946ae16d9c88b7fc85c5a53960ead6ac56646ec5c593cb
release architecture multi
default architecture amd64

%> oc version
Client Version: 5.0.0-ec.5
Kustomize Version: v5.8.1
Kubernetes Version: v1.36.2
```

### It's a compact

Just used my regular automation to deploy a compact cluster, it just worked seemingly and smoothly.

```
%> bash/cluster.sh list
INFRA_ID: lab-wf7n2

-------------------------------------------------------------------------------------------
|                                    DescribeInstances                                    |
+---------------+-----------------------+-----------------------+--------------+----------+
|      AZ       |          ID           |         Name          |  PrivateIP   |  State   |
+---------------+-----------------------+-----------------------+--------------+----------+
|  eu-central-1b|  i-01f4698fc401173ae  |  lab-wf7n2-master-1   |  10.0.41.129 |  running |
|  eu-central-1c|  i-00d075ecd138c4d66  |  lab-wf7n2-master-2   |  10.0.70.213 |  running |
|  eu-central-1a|  i-0122a0c13c95f5f90  |  lab-wf7n2-bootstrap  |  10.0.96.31  |  running |
|  eu-central-1a|  i-0627f8e6e2e7adb8e  |  lab-wf7n2-master-0   |  10.0.4.78   |  running |
+---------------+-----------------------+-----------------------+--------------+----------+
```

```
%> bash/create-cluster.sh
INFO ipFamily is not specified in install-config; defaulting to "IPv4"
INFO Credentials loaded from the AWS config using "SharedConfigCredentials: ~/.aws/credentials" provider
WARNING Making control-plane schedulable by setting MastersSchedulable to true for Scheduler cluster settings
INFO Successfully populated MCS CA cert information: root-ca 2036-08-09T12:37:13Z 2026-08-12T12:37:13Z
INFO Successfully populated MCS TLS cert information: root-ca 2036-08-09T12:37:13Z 2026-08-12T12:37:13Z
INFO Consuming Install Config from target directory
INFO Adding clusters...
INFO Creating infrastructure resources...
INFO Reconciling IAM roles for control-plane and compute nodes
INFO Creating IAM role for master
INFO Creating IAM role for worker
INFO Started local control plane with envtest
INFO Stored kubeconfig for envtest in: ~/demos/ocp-on-aws/ipi/.clusterapi_output/envtest.kubeconfig
INFO Running process: Cluster API with args [-v=2 --diagnostics-address=0 --health-addr=127.0.0.1:35435 --webhook-port=41775 --webhook-cert-dir=/tmp/envtest-serving-certs-1877808257 --kubeconfig=~/demos/ocp-on-aws/ipi/.clusterapi_output/envtest.kubeconfig]
INFO Running process: aws infrastructure provider with args [-v=4 --diagnostics-address=0 --health-addr=127.0.0.1:46621 --webhook-port=42085 --webhook-cert-dir=/tmp/envtest-serving-certs-4223505619 --feature-gates=BootstrapFormatIgnition=true,ExternalResourceGC=true,TagUnmanagedNetworkResources=false,EKS=false,MachinePool=false --kubeconfig=~/demos/ocp-on-aws/ipi/.clusterapi_output/envtest.kubeconfig]
INFO Creating infra manifests...
INFO Created manifest *v1.Namespace, namespace= name=openshift-cluster-api-guests
INFO Created manifest *v1beta2.AWSClusterControllerIdentity, namespace= name=default
I0812 14:37:46.543646 1434068 warning_handler.go:65] "cluster.x-k8s.io/v1beta1 Cluster is deprecated; use cluster.x-k8s.io/v1beta2 Cluster" logger="KubeAPIWarningLogger"
INFO Created manifest *v1beta1.Cluster, namespace=openshift-cluster-api-guests name=lab-wf7n2
INFO Created manifest *v1beta2.AWSCluster, namespace=openshift-cluster-api-guests name=lab-wf7n2
INFO Done creating infra manifests
INFO Creating kubeconfig entry for capi cluster lab-wf7n2
INFO Waiting up to 15m0s (until 2:52PM CEST) for network infrastructure to become ready...
INFO Network infrastructure is ready
INFO Creating Route53 records for control plane load balancer
INFO Created private Hosted Zone
INFO Created manifest *v1beta2.AWSMachine, namespace=openshift-cluster-api-guests name=lab-wf7n2-bootstrap
INFO Created manifest *v1beta2.AWSMachine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-0
INFO Created manifest *v1beta2.AWSMachine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-1
INFO Created manifest *v1beta2.AWSMachine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-2
I0812 14:42:48.741749 1434068 warning_handler.go:65] "cluster.x-k8s.io/v1beta1 Machine is deprecated; use cluster.x-k8s.io/v1beta2 Machine" logger="KubeAPIWarningLogger"
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=lab-wf7n2-bootstrap
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-0
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-1
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-2
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=lab-wf7n2-bootstrap
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=lab-wf7n2-master
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=lab-wf7n2-worker
INFO Waiting up to 15m0s (until 2:52PM CEST) for network infrastructure to become ready...
INFO Network infrastructure is ready
INFO Creating Route53 records for control plane load balancer
INFO Created private Hosted Zone
INFO Created manifest *v1beta2.AWSMachine, namespace=openshift-cluster-api-guests name=lab-wf7n2-bootstrap
INFO Created manifest *v1beta2.AWSMachine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-0 
INFO Created manifest *v1beta2.AWSMachine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-1 
INFO Created manifest *v1beta2.AWSMachine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-2 
I0812 14:42:48.741749 1434068 warning_handler.go:65] "cluster.x-k8s.io/v1beta1 Machine is deprecated; use cluster.x-k8s.io/v1beta2 Machine" logger="KubeAPIWarningLogger"
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=lab-wf7n2-bootstrap
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-0 
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-1 
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=lab-wf7n2-master-2 
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=lab-wf7n2-bootstrap 
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=lab-wf7n2-master 
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=lab-wf7n2-worker 
INFO Waiting up to 15m0s (until 2:57PM CEST) for machines [lab-wf7n2-bootstrap lab-wf7n2-master-0 lab-wf7n2-master-1 lab-wf7n2-master-2] to provision... 
INFO Control-plane machines are ready             
INFO Cluster API resources have been created. Waiting for cluster to become ready... 
INFO Waiting up to 20m0s (until 3:03PM CEST) for the Kubernetes API at https://api.lab.example.com:6443... 
INFO API v1.36.2 up                               
INFO Waiting up to 45m0s (until 3:42PM CEST) for bootstrapping to complete... 
INFO Waiting for the bootstrap etcd member to be removed... 
INFO Bootstrap etcd member has been removed       
INFO Destroying the bootstrap resources...        
INFO Waiting up to 5m0s for bootstrap machine deletion openshift-cluster-api-guests/lab-wf7n2-bootstrap... 
INFO Shutting down local Cluster API controllers...
INFO Stopped controller: Cluster API              
INFO Stopped controller: aws infrastructure provider 
INFO Shutting down local Cluster API control plane... 
INFO Local Cluster API system has completed operations 
INFO Finished destroying bootstrap resources      
INFO Waiting up to 40m0s (until 3:46PM CEST) for the cluster at https://api.lab.example.com:6443 to initialize... 
INFO Waiting up to 30m0s (until 3:41PM CEST) to ensure each cluster operator has finished progressing... 
INFO All cluster operators have completed progressing 
INFO Checking to see if there is a route at openshift-console/console... 
INFO Install complete!                            
INFO To access the cluster as the system:admin user when using 'oc', run 
INFO     export KUBECONFIG=~/demos/ocp-on-aws/ipi/auth/kubeconfig 
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.lab.example.com 
INFO Login to the console with user: "kubeadmin", and password: "secret" 
INFO Time elapsed: 41m1s                          
%> 
```

```
%> oc get clusterversion,co
NAME                                         VERSION      AVAILABLE   PROGRESSING   SINCE   STATUS
clusterversion.config.openshift.io/version   5.0.0-ec.5   True        False         6m48s   Cluster version is 5.0.0-ec.5

NAME                                                                           VERSION      AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
clusteroperator.config.openshift.io/authentication                             5.0.0-ec.5   True        False         False      8m47s
clusteroperator.config.openshift.io/baremetal                                  5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/cloud-controller-manager                   5.0.0-ec.5   True        False         False      26m
clusteroperator.config.openshift.io/cloud-credential                           5.0.0-ec.5   True        False         False      12m
clusteroperator.config.openshift.io/cluster-autoscaler                         5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/config-operator                            5.0.0-ec.5   True        False         False      25m
clusteroperator.config.openshift.io/console                                    5.0.0-ec.5   True        False         False      8m41s
clusteroperator.config.openshift.io/control-plane-machine-set                  5.0.0-ec.5   True        False         False      23m
clusteroperator.config.openshift.io/csi-snapshot-controller                    5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/dns                                        5.0.0-ec.5   True        False         False      23m
clusteroperator.config.openshift.io/etcd                                       5.0.0-ec.5   True        False         False      22m
clusteroperator.config.openshift.io/image-registry                             5.0.0-ec.5   True        False         False      11m
clusteroperator.config.openshift.io/ingress                                    5.0.0-ec.5   True        False         False      12m
clusteroperator.config.openshift.io/insights                                   5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/kube-apiserver                             5.0.0-ec.5   True        False         False      18m
clusteroperator.config.openshift.io/kube-controller-manager                    5.0.0-ec.5   True        False         False      22m
clusteroperator.config.openshift.io/kube-scheduler                             5.0.0-ec.5   True        False         False      22m
clusteroperator.config.openshift.io/kube-storage-version-migrator              5.0.0-ec.5   True        False         False      25m
clusteroperator.config.openshift.io/machine-api                                5.0.0-ec.5   True        False         False      20m
clusteroperator.config.openshift.io/machine-approver                           5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/machine-config                             5.0.0-ec.5   True        False         False      25m
clusteroperator.config.openshift.io/marketplace                                5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/monitoring                                 5.0.0-ec.5   True        False         False      7m15s
clusteroperator.config.openshift.io/network                                    5.0.0-ec.5   True        False         False      26m
clusteroperator.config.openshift.io/node-tuning                                5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/olm                                        5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/openshift-apiserver                        5.0.0-ec.5   True        False         False      12m
clusteroperator.config.openshift.io/openshift-controller-manager               5.0.0-ec.5   True        False         False      21m
clusteroperator.config.openshift.io/openshift-samples                          5.0.0-ec.5   True        False         False      11m
clusteroperator.config.openshift.io/operator-lifecycle-manager                 5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/operator-lifecycle-manager-catalog         5.0.0-ec.5   True        False         False      24m
clusteroperator.config.openshift.io/operator-lifecycle-manager-packageserver   5.0.0-ec.5   True        False         False      12m
clusteroperator.config.openshift.io/service-ca                                 5.0.0-ec.5   True        False         False      25m
clusteroperator.config.openshift.io/storage                                    5.0.0-ec.5   True        False         False      23m
```

## Installing ODF Operator

My automation failed here, so it's my first stop.

```
%> scripts/00-preflight.sh 
Error from server (NotFound): packagemanifests.packages.operators.coreos.com "odf-operator" not found
Error from server (NotFound): packagemanifests.packages.operators.coreos.com "odf-operator" not found
```

# Conclusions Up To Now (2026-08-12)

* Current IPI Installation method is backwards compatible, no surprises here.
* ODF Operator installation automation failed.