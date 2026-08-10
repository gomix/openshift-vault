
# OpenShift Insights

**Red Hat Insights for OpenShift** is a remote health monitoring and analysis service designed to proactively identify potential risks and issues affecting OpenShift clusters.

Within an OpenShift cluster, the **Insights Operator** collects selected configuration and operational data and periodically sends it to Red Hat for analysis. The service can then use this information to identify known risks related to areas such as availability, performance, fault tolerance, and security.

The Insights Operator runs as part of the OpenShift `Insights` cluster capability and complements OpenShift Telemetry.


```
OpenShift
   │
   └── Insights Operator
          │
          │ collects cluster data
          ▼
      Red Hat Insights
          │
          └── Advisor
                │
                ▼
          Recommendations
```


```
%> oc get co insights
NAME       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
insights   4.18.25   True        False         False      11m   

%> oc get pods -n openshift-insights
NAME                                 READY   STATUS    RESTARTS        AGE
insights-operator-7dcf5bd85b-hszpj   1/1     Running   1 (8m28s ago)   11m
```

What kind of information about my cluster and how i can get it?

### events


```bash
%> oc get event --sort-by=lastTimestamp
LAST SEEN   TYPE      REASON                                OBJECT                                              MESSAGE
18m         Warning   KubeAPIReadyz                         namespace/openshift-kube-apiserver                  readyz=true
16m         Normal    NodeHasSufficientPID                  node/ip-10-0-79-220.eu-central-1.compute.internal   Node ip-10-0-79-220.eu-central-1.compute.internal status is now: NodeHasSufficientPID         
16m         Normal    NodeHasNoDiskPressure                 node/ip-10-0-79-220.eu-central-1.compute.internal   Node ip-10-0-79-220.eu-central-1.compute.internal status is now: NodeHasNoDiskPressure        
16m         Normal    NodeHasSufficientMemory               node/ip-10-0-79-220.eu-central-1.compute.internal   Node ip-10-0-79-220.eu-central-1.compute.internal status is now: NodeHasSufficientMemory      
16m         Normal    RegisteredNode                        node/ip-10-0-79-220.eu-central-1.compute.internal   Node ip-10-0-79-220.eu-central-1.compute.internal event: Registered Node ip-10-0-79-220.eu-central-1.compute.internal in Controller
15m         Normal    NodeHasSufficientMemory               node/ip-10-0-38-95.eu-central-1.compute.internal    Node ip-10-0-38-95.eu-central-1.compute.internal status is now: NodeHasSufficientMemory       
15m         Normal    NodeHasSufficientPID                  node/ip-10-0-38-95.eu-central-1.compute.internal    Node ip-10-0-38-95.eu-central-1.compute.internal status is now: NodeHasSufficientPID
15m         Normal    NodeHasNoDiskPressure                 node/ip-10-0-38-95.eu-central-1.compute.internal    Node ip-10-0-38-95.eu-central-1.compute.internal status is now: NodeHasNoDiskPressure         
15m         Normal    NodeHasNoDiskPressure                 node/ip-10-0-20-252.eu-central-1.compute.internal   Node ip-10-0-20-252.eu-central-1.compute.internal status is now: NodeHasNoDiskPressure        
15m         Normal    NodeHasSufficientMemory               node/ip-10-0-20-252.eu-central-1.compute.internal   Node ip-10-0-20-252.eu-central-1.compute.internal status is now: NodeHasSufficientMemory      
15m         Normal    RegisteredNode                        node/ip-10-0-20-252.eu-central-1.compute.internal   Node ip-10-0-20-252.eu-central-1.compute.internal event: Registered Node ip-10-0-20-252.eu-central-1.compute.internal in Controller
15m         Normal    RegisteredNode                        node/ip-10-0-38-95.eu-central-1.compute.internal    Node ip-10-0-38-95.eu-central-1.compute.internal event: Registered Node ip-10-0-38-95.eu-central-1.compute.internal in Controller
15m         Normal    Synced                                node/ip-10-0-38-95.eu-central-1.compute.internal    Node synced successfully
15m         Normal    Synced                                node/ip-10-0-20-252.eu-central-1.compute.internal   Node synced successfully
15m         Normal    Synced                                node/ip-10-0-79-220.eu-central-1.compute.internal   Node synced successfully
14m         Normal    CSRApproved                           certificatesigningrequest/csr-xtxtd                 CSR "csr-xtxtd" has been approved
14m         Normal    CSRApproved                           certificatesigningrequest/csr-9gc7w                 CSR "csr-9gc7w" has been approved
14m         Normal    CSRApproved                           certificatesigningrequest/csr-cz5ds                 CSR "csr-cz5ds" has been approved
14m         Normal    CSRApproved                           certificatesigningrequest/csr-zv7qs                 CSR "csr-zv7qs" has been approved
14m         Normal    CSRApproved                           certificatesigningrequest/csr-6p4gg                 CSR "csr-6p4gg" has been approved
14m         Normal    CSRApproved                           certificatesigningrequest/csr-fcz84                 CSR "csr-fcz84" has been approved
14m         Normal    NodeReady                             node/ip-10-0-38-95.eu-central-1.compute.internal    Node ip-10-0-38-95.eu-central-1.compute.internal status is now: NodeReady
13m         Normal    NodeReady                             node/ip-10-0-79-220.eu-central-1.compute.internal   Node ip-10-0-79-220.eu-central-1.compute.internal status is now: NodeReady
9m2s        Normal    Status upgrade                        clusteroperator/machine-api                         Progressing towards operator: 4.18.25
3m19s       Normal    ShutdownInitiated                     namespace/openshift-kube-apiserver                  Received signal to terminate, becoming unready, but keeping serving
3m19s       Normal    TerminationPreShutdownHooksFinished   namespace/openshift-kube-apiserver                  All pre-shutdown hooks have been finished
2m56s       Normal    RegisteredNode                        node/ip-10-0-79-220.eu-central-1.compute.internal   Node ip-10-0-79-220.eu-central-1.compute.internal event: Registered Node ip-10-0-79-220.eu-central-1.compute.internal in Controller
2m56s       Normal    RegisteredNode                        node/ip-10-0-20-252.eu-central-1.compute.internal   Node ip-10-0-20-252.eu-central-1.compute.internal event: Registered Node ip-10-0-20-252.eu-central-1.compute.internal in Controller
2m56s       Normal    RegisteredNode                        node/ip-10-0-38-95.eu-central-1.compute.internal    Node ip-10-0-38-95.eu-central-1.compute.internal event: Registered Node ip-10-0-38-95.eu-central-1.compute.internal in Controller`

```