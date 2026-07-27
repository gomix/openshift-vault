---
tags:
  - installation
  - ipi
  - aws
  - recovery
---

Your `openshift-install create cluster` ends like this:

```shell
INFO Destroying the bootstrap resources...                                                                                                                                        
FATAL error destroying bootstrap resources failed during the destroy bootstrap hook: unable to remove bootstrap ssh rule: error getting security group: RequestError: send request
 failed                                                                                                                                                                           
FATAL caused by: Post "https://ec2.eu-central-1.amazonaws.com/": read tcp 10.202.145.18:48712->52.94.141.18:443: read: connection reset by peer
INFO Shutting down local Cluster API controllers...                 
INFO Stopped controller: Cluster API                                   
INFO Stopped controller: aws infrastructure provider
INFO Shutting down local Cluster API control plane...       
INFO Local Cluster API system has completed operations  
```

Error message is pretty clear, what to do then? Should restart from zero or can we recover?

- Is the cluster alive?
```
export KUBECONFIG=auth/kubeconfig

oc whoami
oc get nodes
oc get co
```

- Verify the EC2 instances, pay attention specially to the bootstrap EC2 instance.
  
```bash
list_instances() {
    local infra_id="$1"

    aws ec2 describe-instances \
        --filters \
            "Name=tag:kubernetes.io/cluster/${infra_id},Values=owned" \
            "Name=instance-state-name,Values=pending,running,stopping,stopped" \
        --query 'Reservations[].Instances[].{
            Name: Tags[?Key==`Name`] | [0].Value,
            ID: InstanceId,
            State: State.Name,
            AZ: Placement.AvailabilityZone,
            PrivateIP: PrivateIpAddress
        }' \
        --output table
}
```
You can use the Bash function provided in your script where `${infra_id}` is located in your `metada.jso` file in your installation directory to get something like:
```bash
--------------------------------------------------------------------------------------------
|                                     DescribeInstances                                    |
+---------------+-----------------------+-----------------------+---------------+----------+
|      AZ       |          ID           |         Name          |   PrivateIP   |  State   |
+---------------+-----------------------+-----------------------+---------------+----------+
|  eu-central-1a|  i-0e8260d81654a8701  |  lab-kvflr-master-0   |  10.0.23.105  |  running |
|  eu-central-1a|  i-0849b2ad3b6171f5c  |  lab-kvflr-bootstrap  |  10.0.100.185 |  running |
|  eu-central-1b|  i-0f9cec9faeaa14bc3  |  lab-kvflr-master-1   |  10.0.57.139  |  running |
|  eu-central-1c|  i-0628bb6e1b6119628  |  lab-kvflr-master-2   |  10.0.76.20   |  running |
+---------------+-----------------------+-----------------------+---------------+----------+
```
Things you can try:
```
$ openshift-install wait-for install-complete -h
Wait until the cluster is ready

Usage:
  openshift-install wait-for install-complete [flags]

Flags:
  -h, --help   help for install-complete

Global Flags:
      --dir string         assets directory (default ".")
      --log-level string   log level (e.g. "debug | info | warn | error") (default "info")
      
$ openshift-install destroy bootstrap -h
Destroy the bootstrap resources

Usage:
  openshift-install destroy bootstrap [flags]

Flags:
  -h, --help   help for bootstrap

Global Flags:
      --dir string         assets directory (default ".")
      --log-level string   log level (e.g. "debug | info | warn | error") (default "info")

```
## Root Cause

- My laptop went offline before finishing the installation (version 4.18.25) , state of the files in installation directory are not consistent so none of the previous commands helped.
- Escape hatch, go directly to AWS and do a manual cleanup.

In my case i decided to extend my current scripts, code and sample output follows:

```bash
get_bootstrap_instance_id() {                                                                                                                                                     
    local infra_id="$1"                                                                    
    aws ec2 describe-instances \
        --filters \                                                                        
            "Name=tag:kubernetes.io/cluster/${infra_id},Values=owned" \
            "Name=instance-state-name,Values=pending,running,stopping,stopped" \
        --query "Reservations[].Instances[?Tags[?Key=='Name' && contains(Value, 'bootstrap')]].InstanceId" \
        --output text                                                          
}                                                                                                                                                            
cleanup_bootstrap() {
    local infra_id="$1"                                                                 
    local instance_id
    instance_id=$(get_bootstrap_instance_id "$infra_id")
    if [[ -z "$instance_id" || "$instance_id" == "None" ]]; then                            
        echo "Bootstrap instance not found."
        return 0                                          
    fi                                                                                      
    echo "Bootstrap instance found:"
    echo "  InstanceId: $instance_id"                                                  
    echo
    read -rp "Terminate bootstrap instance? [y/N] " answer
    case "$answer" in                                                                  
        y|Y|yes|YES)                            
            ;;                                                                 
        *)                                                                        
            echo "Cancelled."
            return 0                                                         
            ;;                                                                  
    esac
    
    echo                                                                                  
    echo "Terminating bootstrap instance..."
    aws ec2 terminate-instances \                                          
        --instance-ids "$instance_id" \                                                     
        >/dev/null
    echo "Waiting for instance termination..."
    aws ec2 wait instance-terminated \                                        
        --instance-ids "$instance_id"
    echo                                                 
    echo "Bootstrap instance terminated."
} 
```

```
$ bash/cluster.sh cleanup-bootstrap
INFRA_ID: lab-kvflr

--------------------------------------------------------------------------------------------
|                                     DescribeInstances                                    |
+---------------+-----------------------+-----------------------+---------------+----------+
|      AZ       |          ID           |         Name          |   PrivateIP   |  State   |
+---------------+-----------------------+-----------------------+---------------+----------+
|  eu-central-1a|  i-0e8260d81654a8701  |  lab-kvflr-master-0   |  10.0.23.105  |  running |
|  eu-central-1a|  i-0849b2ad3b6171f5c  |  lab-kvflr-bootstrap  |  10.0.100.185 |  running |
|  eu-central-1b|  i-0f9cec9faeaa14bc3  |  lab-kvflr-master-1   |  10.0.57.139  |  running |
|  eu-central-1c|  i-0628bb6e1b6119628  |  lab-kvflr-master-2   |  10.0.76.20   |  running |
+---------------+-----------------------+-----------------------+---------------+----------+

Bootstrap instance found:
  InstanceId: i-0849b2ad3b6171f5c

Terminate bootstrap instance? [y/N] y

Terminating bootstrap instance...
Waiting for instance termination...

Bootstrap instance terminated.

------------------------------------------------------------------------------------------
|                                    DescribeInstances                                   |
+---------------+-----------------------+---------------------+--------------+-----------+
|      AZ       |          ID           |        Name         |  PrivateIP   |   State   |
+---------------+-----------------------+---------------------+--------------+-----------+
|  eu-central-1a|  i-0e8260d81654a8701  |  lab-kvflr-master-0 |  10.0.23.105 |  running  |
|  eu-central-1b|  i-0f9cec9faeaa14bc3  |  lab-kvflr-master-1 |  10.0.57.139 |  running  |
|  eu-central-1c|  i-0628bb6e1b6119628  |  lab-kvflr-master-2 |  10.0.76.20  |  running  |
+---------------+-----------------------+---------------------+--------------+-----------+
```

Follow up, is the installation really clean? (TODO)