## Deployment

[https://kubernetes.io/docs/concepts/workloads/controllers/deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

## Deployment Strategies

- RollingUpdate
  - In this strategy, both versions of the application run simultaneously, and it scales down instances of the previous version only when the new version is ready. The main drawback is that this strategy requires compatibility between the versions in the deployment.
  - The RollingUpdate strategy is the default strategy if you do not specify a strategy on the Deployment objects.
- Recreate
  - You can use this strategy when your application cannot have different simultaneously running code versions.

## Create Deployment

```
1 $ oc create deployment db-pod --port 3306 \
2 > --image registry.ocp4.example.com:8443/rhel8/mysql-80
3 deployment.apps/db-pod created
4
5 $ oc set env deployment/db-pod \
6 > MYSQL_USER=user1 \
7 > MYSQL_PASSWORD=mypa55w0rd \
8 > MYSQL_DATABASE=items
9 deployment.apps/db-pod updated
10
11 ; create and set PVC for the deployment
12 $ oc set volumes deployment/db-pod \
13 > --add --name lvm-storage --type pvc \
14 > --claim-mode rwo --claim-size 1Gi --mount-path /var/lib/mysql \
15 > --claim-class lvms-vg1 \
16 > --claim-name db-pod-pvc
17 deployment.apps/db-pod volume updated
18
```

## Sample Deployment

```
1 ---
2 apiVersion: apps/v1
3 kind: Deployment
4 metadata:
5 labels:
6 app: database
7 name: database
8 spec:
9 replicas: 1
10 selector:
11 matchLabels:
12 app: database
13 strategy:
14 type: Recreate
15 template:
16 metadata:
17 labels:
18 app: database
19 name: database
20 spec:
21 containers:
22 - env:
23 - name: POSTGRESQL_DATABASE
24 valueFrom:
```

```
25 secretKeyRef:
26 key: database-name
27 name: database
28 - name: POSTGRESQL_USER
29 valueFrom:
30 secretKeyRef:
31 key: database-user
32 name: database
33 - name: POSTGRESQL_PASSWORD
34 valueFrom:
35 secretKeyRef:
36 key: database-password
37 name: database
38 - name: POSTGRESQL_ADMIN_PASSWORD
39 valueFrom:
40 secretKeyRef:
41 key: database-admin-password
42 name: database
43 envFrom:
44 - configMapRef:
45 name: database
46 image: registry.ocp4.example.com:8443/rhel8/postgresql-13:1-7
47 imagePullPolicy: Always
48 livenessProbe:
49 exec:
50 command:
51 - /usr/libexec/check-container
52 - --live
53 initialDelaySeconds: 120
54 timeoutSeconds: 10
55 name: postgresql
56 ports:
57 - containerPort: 5432
58 protocol: TCP
59 readinessProbe:
60 exec:
61 command:
62 - /usr/libexec/check-container
63 initialDelaySeconds: 5
64 timeoutSeconds: 1
65 resources:
66 limits:
67 cpu: 250m
68 memory: 1Gi
69 requests:
70 cpu: 100m
71 memory: 512Mi
72 securityContext:
73 allowPrivilegeEscalation: false
74 capabilities:
75 drop:
76 - ALL
77 privileged: false
78 runAsNonRoot: true
79 seccompProfile:
80 type: RuntimeDefault
81 terminationMessagePath: /dev/termination-log
82 volumeMounts:
83 - mountPath: /var/lib/pgsql/data
84 name: database-data
85 dnsPolicy: ClusterFirst
86 restartPolicy: Always
87 volumes:
88 - emptyDir:
89 medium: ""
90 name: database-data
91
```