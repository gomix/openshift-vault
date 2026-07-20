# Warp Benchmarking ODF/NooBaa S3
* https://github.com/minio/warp

## Manifests to deploy

### Namespace

`noobaa-benchmark.yaml`

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: noobaa-benchmark
```

### Warp Listener

`warp-client.yaml`

```
---
apiVersion: v1
kind: Pod
metadata:
  name: warp-client
  namespace: noobaa-benchmark
  labels:
    app: warp-client
spec:
  restartPolicy: Never
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: warp
      image: minio/warp:latest                            
      imagePullPolicy: Always
      args:
        - client
        - 0.0.0.0:7761
      ports:
        - name: warp
          containerPort: 7761
          protocol: TCP
      resources:
        requests:
          cpu: 100m
          memory: 128Mi
        limits:
          cpu: "2"
          memory: 2Gi
      securityContext:
        runAsNonRoot: true
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL
```

* * *

### Storage

Target bucket to use for the benchmark tests (read/write).
`obc-1.yaml`

```
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: noobaa-warp-benchmark
  namespace: noobaa-benchmark
spec:
  bucketName: noobaa-warp-benchmark
  storageClassName: openshift-storage.noobaa.io
  additionalConfig:
    bucketclass: noobaa-default-bucket-class
```

PVC to hold results from the benchmarks tests (Jobs).
`warp-results-pvc.yaml`

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: warp-results
  namespace: noobaa-benchmark
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp3-csi                  <<<< ADAPT
  resources:
    requests:
      storage: 1Gi
```

* * *

### Benchmark Tests (Job)

`job-1.yaml`

```
apiVersion: batch/v1
kind: Job
metadata:
  name: noobaa-warp-smoke
  namespace: noobaa-benchmark
spec:
  backoffLimit: 0
  template:
    metadata:
      labels:
        app: noobaa-warp
        test: smoke
    spec:
      restartPolicy: Never
      securityContext:
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: warp
          image: minio/warp:latest
          imagePullPolicy: Always

          args:
            - mixed
            - --host=s3.openshift-storage.svc:443                     <<< S3 service target, ADAPT
            - --tls
            - --insecure
            - --access-key=$(AWS_ACCESS_KEY_ID)
            - --secret-key=$(AWS_SECRET_ACCESS_KEY)
            - --bucket=$(BUCKET_NAME)
            - --duration=1m
            - --concurrent=4
            - --objects=100
            - --obj.size=1MiB
            - --noclear
            - --analyze.v
            - --benchdata=/results/$(WARP_RUN_ID).csv.zst
          env:
            - name: WARP_RUN_ID
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
          
            - name: AWS_ACCESS_KEY_ID
              valueFrom:
                secretKeyRef:
                  name: noobaa-warp-benchmark
                  key: AWS_ACCESS_KEY_ID
          
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: noobaa-warp-benchmark
                  key: AWS_SECRET_ACCESS_KEY
          
            - name: BUCKET_NAME
              valueFrom:
                configMapKeyRef:
                  name: noobaa-warp-benchmark
                  key: BUCKET_NAME

          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "2"
              memory: 2Gi

          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL

          volumeMounts:
            - name: results
              mountPath: /results

      volumes:
        - name: results
          persistentVolumeClaim:
            claimName: warp-results
```

* * *

### Benchmark Results

Warp results reader pod.
`warp-results-reader.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: warp-results-reader
  namespace: noobaa-benchmark
  labels:
    app: warp-results-reader
spec:
  restartPolicy: Never

  securityContext:
    seccompProfile:
      type: RuntimeDefault

  containers:
    - name: reader
      image: registry.access.redhat.com/ubi9/ubi:latest
      imagePullPolicy: IfNotPresent

      command:
        - /bin/bash
        - -c
        - sleep infinity

      resources:
        requests:
          cpu: 10m
          memory: 32Mi
        limits:
          cpu: 100m
          memory: 128Mi

      securityContext:
        runAsNonRoot: true
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL

      volumeMounts:
        - name: results
          mountPath: /results
          readOnly: true

  volumes:
    - name: results
      persistentVolumeClaim:
        claimName: warp-results
```

* * *

## Deploy and sample outputs

```
; namespace
%> oc apply -f namespace.yaml 
namespace/noobaa-benchmark created
%> oc project noobaa-benchmark
Now using project "noobaa-benchmark" on server "https://api.lab.sandbox502.opentlc.com:6443".

; storage components
%> oc apply -f obc-1.yaml 
objectbucketclaim.objectbucket.io/noobaa-warp-benchmark created

%> oc apply -f warp-results-pvc.yaml 
persistentvolumeclaim/warp-results created

; warp listener
%> oc apply -f warp-client.yaml 
pod/warp-client created

; check status
%> oc get obc
NAME                    STORAGE-CLASS                 PHASE   AGE
noobaa-warp-benchmark   openshift-storage.noobaa.io   Bound   2m55s

%> oc get cm
NAME                       DATA   AGE
kube-root-ca.crt           1      16h
noobaa-warp-benchmark      5      2m52s           <<< test bucket data/credentials
openshift-service-ca.crt   1      16h

%> oc get pvc
NAME           STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
warp-results   Pending                                      gp3-csi        <unset>                 2m55s
; pending is normal, will be used by jobs

%> oc get pod
NAME          READY   STATUS    RESTARTS   AGE
warp-client   1/1     Running   0          92s; launch the benchmart test (job)

; benchmark job launch
%> oc apply -f job-1.yaml 
job.batch/noobaa-warp-smoke created%> oc get pod
NAME                      READY   STATUS      RESTARTS   AGE
noobaa-warp-smoke-zxcqg   0/1     Completed   0          2m3s
warp-client               1/1     Running     0          3m56s

; get the logs
%> oc logs pod/noobaa-warp-smoke-zxcqg

Report: DELETE (508 reqs). Ran Duration: 57s, starting 09:24:13 UTC
 * Objects per request: 1. Concurrency: 4.
 * Average: 8.47 obj/s (57s)
 * Reqs: Avg: 13.1ms, 50%: 7.1ms, 90%: 29.9ms, 99%: 93.8ms, Fastest: 4.4ms, Slowest: 125.1ms, StdDev: 17.7ms

Throughput, split into 57 x 1s:
 * Fastest: 16.96 obj/s (1s, starting 09:24:33 UTC)
 * 50% Median: 8.00 obj/s (1s, starting 09:24:42 UTC)
 * Slowest: 3.00 obj/s (1s, starting 09:24:30 UTC)

──────────────────────────────────

Report: GET (2276 reqs). Ran Duration: 57s, starting 09:24:13 UTC
 * Objects per request: 1. Size: 1048576 bytes. Concurrency: 4.
 * Average: 38.36 MiB/s, 38.36 obj/s (57s)
 * Reqs: Avg: 51.5ms, 50%: 28.3ms, 90%: 125.3ms, 99%: 186.5ms, Fastest: 8.1ms, Slowest: 273.7ms, StdDev: 47.7ms
 * TTFB: Avg: 50ms, Best: 7ms, 25th: 17ms, Median: 27ms, 75th: 82ms, 90th: 124ms, 99th: 185ms, Worst: 273ms StdDev: 48ms

Throughput, split into 57 x 1s:
 * Fastest: 62.0MiB/s, 62.00 obj/s (1s, starting 09:24:38 UTC)
 * 50% Median: 37.2MiB/s, 37.24 obj/s (1s, starting 09:24:50 UTC)
 * Slowest: 16.6MiB/s, 16.63 obj/s (1s, starting 09:24:58 UTC)

──────────────────────────────────

Report: PUT (756 reqs). Ran Duration: 57s, starting 09:24:13 UTC
 * Objects per request: 1. Size: 1048576 bytes. Concurrency: 4.
 * Average: 12.85 MiB/s, 12.85 obj/s (57s)
 * Reqs: Avg: 153.7ms, 50%: 141.0ms, 90%: 219.3ms, 99%: 289.8ms, Fastest: 82.7ms, Slowest: 335.8ms, StdDev: 44.8ms

Throughput, split into 57 x 1s:
 * Fastest: 18.2MiB/s, 18.16 obj/s (1s, starting 09:24:24 UTC)
 * 50% Median: 13.2MiB/s, 13.17 obj/s (1s, starting 09:24:38 UTC)
 * Slowest: 6.4MiB/s, 6.39 obj/s (1s, starting 09:25:01 UTC)

──────────────────────────────────

Report: STAT (1520 reqs). Ran Duration: 57s, starting 09:24:13 UTC
 * Objects per request: 1. Concurrency: 4.
 * Average: 25.44 obj/s (57s)
 * Reqs: Avg: 6.1ms, 50%: 3.1ms, 90%: 11.6ms, 99%: 65.4ms, Fastest: 2.0ms, Slowest: 108.5ms, StdDev: 10.3ms

Throughput, split into 57 x 1s:
 * Fastest: 40.00 obj/s (1s, starting 09:24:27 UTC)
 * 50% Median: 25.49 obj/s (1s, starting 09:24:37 UTC)
 * Slowest: 12.00 obj/s (1s, starting 09:25:06 UTC)

──────────────────────────────────

Report: Total (5060 reqs). Ran Duration: 57s, starting 09:24:13 UTC
 * Objects per request: 1. Size: 628316 bytes. Concurrency: 4.
 * Average: 51.21 MiB/s, 85.12 obj/s (57s)

Throughput, split into 57 x 1s:
 * Fastest: 75.2MiB/s, 122.69 obj/s (1s, starting 09:24:38 UTC)
 * 50% Median: 55.9MiB/s, 83.88 obj/s (1s, starting 09:24:35 UTC)
 * Slowest: 32.4MiB/s, 53.48 obj/s (1s, starting 09:24:58 UTC)

%> oc get job
NAME                STATUS     COMPLETIONS   DURATION   AGE
noobaa-warp-smoke   Complete   1/1           79s        2m14s

; to run the same benchmark test, you need to recreate the job
%> oc delete job noobaa-warp-smoke
job.batch "noobaa-warp-smoke" deleted

%> oc apply -f job-1.yaml 
job.batch/noobaa-warp-smoke created
```

## Using Result Reader Pod

```
%> oc apply -f warp-results-reader.yaml 
pod/warp-results-reader created

%> oc exec -n noobaa-benchmark warp-results-reader   -- ls -lht /results
total 40K
-rw-r--r--. 1 1000740000 1000740000 9.5K Jul 19 09:35 noobaa-warp-smoke-p8rzf.csv.zst.json.zst
-rw-rw-r--. 1 1000740000 1000740000 9.8K Jul 19 09:25 noobaa-warp-smoke-zxcqg.csv.zst.json.zst
drwxrws---. 2 root       1000740000  16K Jul 19 09:24 lost+found

```

## Analyzing Results

```
%> oc apply -f warp-analyzer.yaml 
pod/warp-analyzer created

%> oc logs warp-analyzer \
  -n noobaa-benchmark
warp: Listening on 127.0.0.1:7761 Press Ctrl+C to exit.

; run warp analyze in warp-analyzer pod manually to get a taste 
%> oc rsh warp-analyzer
~ $ /warp analyze \
> --analyze.op=PUT \                                          < WE ARE INTERESTED IN PUT OPERATIONS
> --analyze.v \
> /results/noobaa-warp-smoke-p8rzf.csv.zst.json.zst           < FROM THIS BENCHMARK TEST
Loading "/results/noobaa-warp-smoke-p8rzf.csv.zst.json.zst"

Report: PUT (702 reqs). Ran Duration: 56s, starting 09:34:37 UTC
 * Objects per request: 1. Size: 1048576 bytes. Concurrency: 4.
 * Average: 11.76 MiB/s, 11.76 obj/s (56s)
 * Reqs: Avg: 166.7ms, 50%: 168.9ms, 90%: 220.5ms, 99%: 328.5ms, Fastest: 81.9ms, Slowest: 411.1ms, StdDev: 47.1ms

Throughput, split into 56 x 1s:
 * Fastest: 16.1MiB/s, 16.09 obj/s (1s, starting 09:35:11 UTC)
 * 50% Median: 12.4MiB/s, 12.38 obj/s (1s, starting 09:35:15 UTC)
 * Slowest: 6.0MiB/s, 5.98 obj/s (1s, starting 09:34:39 UTC)

──────────────────────────────────

Report: Total (4651 reqs). Ran Duration: 57s, starting 09:34:36 UTC
 * Objects per request: 1. Size: 628784 bytes. Concurrency: 4.
 * Average: 46.86 MiB/s, 77.80 obj/s (57s)

Throughput, split into 57 x 1s:
 * Fastest: 58.0MiB/s, 108.63 obj/s (1s, starting 09:34:42 UTC)
 * 50% Median: 42.5MiB/s, 76.11 obj/s (1s, starting 09:35:00 UTC)
 * Slowest: 35.2MiB/s, 48.24 obj/s (1s, starting 09:35:13 UTC)

~ $ 

```

* * *

## Understanding the results

### Test performed

```
A mixed S3 workload was executed against the NooBaa endpoint for approximately one minute.
```

The test used four concurrent operations and combined:

```
Object uploads (PUT)
Object downloads (GET)
Object metadata requests (STAT)
Object deletions (DELETE)
```

Objects used for PUT operations were 1 MiB in size.

```
Test type: mixed
Duration: approximately 57 seconds
Concurrency: 4
PUT object size: 1 MiB
Total operations: 4,651
PUT operations: 702
```

### PUT results

```
During the test, Warp completed 702 PUT operations in 56 seconds.
Average throughput: 11.76 MiB/s
Average rate: 11.76 objects/s
Average latency: 166.7 ms
p50 latency: 168.9 ms
p90 latency: 220.5 ms
p99 latency: 328.5 ms
Fastest PUT: 81.9 ms
Slowest PUT: 411.1 ms
```

Because each object was exactly 1 MiB:

```
11.76 objects/s = 11.76 MiB/s
```

### Reading the latency values

Latency represents how long NooBaa took to complete each PUT operation.

| Metric  | Meaning                                       |
| ------- | --------------------------------------------- |
| Average | Average completion time across all operations |
| p50     | 50% of operations completed within this time  |
| p90     | 90% completed within this time                |
| p99     | 99% completed within this time                |
| Fastest | Fastest operation observed                    |
| Slowest | Slowest operation observed                    |
|StdDev         | Variation in latency around the average                                              |

```
Applied to these results:
Out of every 100 PUT operations, approximately 50 completed in 168.9 ms or less.
Approximately 90 completed in 220.5 ms or less.
Approximately 99 completed in 328.5 ms or less.
The slowest operation took 411.1 ms.
```

### Reading the throughput values

```
Average: 11.76 MiB/s, 11.76 obj/s
```

This means that during the mixed workload:

```
NooBaa completed an average of 11.76 PUT operations per second.
Because each object was 1 MiB, PUT traffic averaged 11.76 MiB/s.
```

Warp also divides throughput into one-second intervals:

```
Fastest second:    16.1 MiB/s
Median second:     12.4 MiB/s
Slowest second:     6.0 MiB/s
```

These values show how throughput varied during the run instead of presenting only a final average.

### Understanding the Total report

```
Report: Total (4,651 requests)
Average: 46.86 MiB/s, 77.80 objects/s
```

`Total` includes every operation in the mixed benchmark:

```
PUT + GET + STAT + DELETE
```

It should therefore not be compared directly with the PUT result of `11.76 MiB/s`.

```
The Total report indicates that Warp processed:
4,651 S3 requests.
77.80 operations per second.
46.86 MiB/s across all data-transfer operations.
```

### Understanding concurrency

```
Concurrency: 4
```

This means Warp maintained up to four simultaneous S3 operations.

```
It does not mean four continuous PUT operations because this was a mixed test. Those four concurrent workers were shared across PUT, GET, STAT, and DELETE operations.
```

## Downloading Results

Just copy files to your ws or reuse the PVC in your own pod/container.

```
%> oc cp ...
```

