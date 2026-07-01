# Secret

A secret is a namespaced object and it can store any type of data.

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in a container image. Using a Secret means that you don't need to include confidential data in your application code.

### Types of Secrets

Kubernetes and OpenShift support the following types of secrets:

- Opaque secrets:
  - An opaque secret store key and value pairs that contain arbitrary values, and are not validated to conform to any convention for key names or values.
- Service account tokens:
  - Store a token credential for applications that authenticate to the Kubernetes API.
- Basic authentication secrets:
  - Store the needed credentials for basic authentication.
  - The data parameter of the secret object must contain the user and the password keys that are encoded in the Base64 format.
- SSH keys:
  - Store data that is used for SSH authentication.
- TLS certificates:
  - Store a certificate and a key that are used for TLS.
- Docker configuration secrets:
  - Store the credentials for accessing a container image registry.

### Creating Secrets

```
1 $ oc create secret generic secret_name \
2 --from-literal key1=secret1 \
3 --from-literal key2=secret2
4
5 $ oc create secret generic ssh-keys \
6 --from-file id_rsa=/path-to/id_rsa \
7 --from-file id_rsa.pub=/path-to/id_rsa.pub
8
9 $ oc create secret tls secret-tls \
10 --cert /path-to-certificate --key /path-to-key
11
12 $ oc create secret docker-registry my-docker-secret \
13 --docker-server=<docker-registry-server> \
14 --docker-username=<your-username> \
15 --docker-password=<your-password> \
16 --docker-email=<your-email>
```

## Using Secrets

#### Secrets to Initialize Environment Variables

The following snippet shows an example of a pod that populates environment variables with data from the testsecret :

```
1 apiVersion: v1
2 kind: Pod
3 metadata:
4 name: secret-example-pod
5 spec:
6 containers:
7 - name: secret-test-container
8 image: busybox
9 command: [ "/bin/sh", "-c", "export" ]
10 env:
11 - name: TEST_SECRET_USERNAME_ENV_VAR
12 valueFrom:
13 secretKeyRef:
14 name: test-secret
15 key: username
```

#### Secrets as Volumes

```
1 $ oc create secret generic demo-secret \
2 --from-literal user=demo-user \
3 --from-literal root_password=zT1KTgk
4
5 $ oc set volume deployment/demo \
6 --add --type secret \
7 --secret-name demo-secret \
8 --mount-path /app-secrets
```

The content of the /app-secrets/user file is demo-user. The content of the /app/secrets/root\_password file is zT1KTgk.

#### References

- <https://kubernetes.io/docs/concepts/configuration/secret/>
- [https://access.redhat.com/documentation/en-us/openshift\\_container\\_platform/4.12/html](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.12/html-single/getting_started/index#getting-started-web-console-creating-secret_openshift-web-console)[single/getting\\_started/index#getting-started-web-console-creating-secret\\_openshift-web-console](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.12/html-single/getting_started/index#getting-started-web-console-creating-secret_openshift-web-console)

```
1 ; using cli
2 oc create secret generic -n openshift-logging loki-secret --from-file=tls.key=<your_key_file> --from-
   file=tls.crt=<your_crt_file> --from-file=ca-bundle.crt=<your_bundle_file> --from-literal=username=<your_username>
   --from-literal=password=<your_password>
3
4 ; declarative yaml
5 ---
6 apiVersion: v1
7 kind: Secret
8 metadata:
9 name: loki-secret
10 namespace: openshift-logging
11
12
```

```
1 apiVersion: v1
2 kind: Secret
3 metadata:
4 name: example-secret
5 namespace: my-app
```

```
6 type: Opaque 7 data: 8 username: bXl1c2VyCg== 9 password: bXlQQDU1Cg== 10 stringData: 11 hostname: myapp.mydomain.com 12 secret.properties: | 13 property1=valueA 14 property2=valueB
```