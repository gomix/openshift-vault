
TBD

```
%> oc describe noobaa noobaa -n openshift-storage | grep -A 52 Welcome 

  Welcome to NooBaa!
  -----------------
  NooBaa Core Version:     5.18.8-9cbd402
  NooBaa Operator Version: 5.18.8

  Lets get started:

  Test S3 client:

    kubectl port-forward -n openshift-storage service/s3 10443:443 &
    NOOBAA_ACCESS_KEY=$(kubectl get secret noobaa-admin -n openshift-storage -o json | jq -r '.data.AWS_ACCESS_KEY_ID|@base64d')
    NOOBAA_SECRET_KEY=$(kubectl get secret noobaa-admin -n openshift-storage -o json | jq -r '.data.AWS_SECRET_ACCESS_KEY|@base64d')
    alias s3='AWS_ACCESS_KEY_ID=$NOOBAA_ACCESS_KEY AWS_SECRET_ACCESS_KEY=$NOOBAA_SECRET_KEY aws --endpoint https://localhost:10443 --no-verify-ssl s3'
    s3 ls


  Services:
    Service Mgmt:
      External DNS:
        https://noobaa-mgmt-openshift-storage.apps.lab.example.com:443
      Internal DNS:
        https://noobaa-mgmt.openshift-storage.svc:443
      Internal IP:
        https://172.30.128.228:443
      Node Ports:
        https://10.0.72.86:0
      Pod Ports:
        https://10.128.0.112:8443
    serviceS3:
      External DNS:
        https://s3-openshiexample.example.com:443
        https://aded7f88dae284256a301162957180fb-1539670292.eu-central-1.elb.amazonaws.com:443
      Internal DNS:
        https://s3.openshift-storage.svc:443
      Internal IP:
        https://172.30.202.41:443
      Node Ports:
        https://10.0.22.189:31545
      Pod Ports:
        https://10.129.0.115:6443
    Service Sts:
      External DNS:
        https://sts-openshift-storage.apps.lab.example.com:443
        https://a6e8463ed54364edcb7a3c3eb29b6a10-536975543.eu-central-1.elb.amazonaws.com:443
      Internal DNS:
        https://sts.openshift-storage.svc:443
      Internal IP:
        https://172.30.48.142:443
      Node Ports:
        https://10.0.22.189:30239
      Pod Ports:
        https://10.129.0.115:7443
    Service Syslog:


%> oc get backingstore -n openshift-storage
NAME                           TYPE     PHASE   AGE
noobaa-default-backing-store   aws-s3   Ready   45m

```