# Managing Kubernetes Secrets

## What is a Secret?

A secret is a critical component for securely managing sensitive data in a Kubernetes environment.

- Secrets are encoded as base64 to handle binary data as text, and are stored unencrypted in the API server's underlying data store (etcd).
- Secrets can be mounted as data volumes or exposed as environment variables to be used by a container in a Pod.
- Individual Secrets are limited to 1MiB in size.
- Secrets can be marked as immutable, preventing changes to the data, by settings `immutable` field to `true`.

## Types of Secret

When creating a Secret, you must specify it's type, which is used to facilitate programmatic handling of Secret data:
```sh
> kubectl create secret <TYPE> ...
```

```yaml
apiVersion: v1
kind: Secret
type: <TYPE>
...
```

| Built-in | Type	Usage |
| - | - |
| Opaque | 	arbitrary user-defined data |
| kubernetes.io/service-account-token |	ServiceAccount token
| kubernetes.io/dockercfg	| serialized ~/.dockercfg file
| kubernetes.io/dockerconfigjson |	serialized ~/.docker/config.json file
| kubernetes.io/basic-auth |	credentials for basic authentication
kubernetes.io/ssh-auth	| credentials for SSH authentication
kubernetes.io/tls	| data for a TLS client or server
bootstrap.kubernetes.io/token |	bootstrap token data

## Create a pod to play around

```sh
kubectl run test --image busybox --dry-run=client -o yaml --command -- sh -c "sleep 3600" > testpod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test
  name: test
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 3600
    image: busybox
    name: test
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Using Secrets as environment variables

You can consume the data in Secrets as environment variable in your container.
- For each container, add an environment variable for each Secret key you want to use, e.g: `containers[].env[].valueFrom.secretKeyRef`

```sh
> kubectl create secret generic stripe-api-key --from-literal=api-key=abc123def456
secret/stripe-api-key created
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test
  name: test
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 3600
    image: busybox
    name: test
    env:
    - name: STRIPE_API_KEY
      valueFrom:
        secretKeyRef:
          name: stripe-api-key
          key: api-key
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```sh
> kubectl replace -f testpod.yaml --force --grace-period=0
pod "test" deleted
pod/test replaced

> kubectl exec test -- env | grep STRIPE
STRIPE_API_KEY=abc123def456
```

>[!IMPORTANT]
>Secret update will not be seen by the container unless it is restarted.
>Secret must exist before it can be referenced, otherwise container will fail with `CreateContainerConfigError`.
>
>

## Using Secrets as volume

You can also consume the data in Secrets as mounted volume in your container, where each top-level key will appear as file.
- In a pod, add volumes with name of the Secret, e.g: `volumes[].secret.secretName`
- For each container, add volume mounts for the volume and mountPath, e.g: `containers[].volumeMounts.[name|mountPath]`

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test
  name: test
spec:
  volumes:
  - name: stripe-api-key
    secret:
      secretName: stripe-api-key
  containers:
  - command:
    - sh
    - -c
    - sleep 3600
    image: busybox
    name: test
    volumeMounts:
    - name: stripe-api-key
      mountPath: /etc/stripe-api-key
      readOnly: true
    env:
    - name: STRIPE_API_KEY
      valueFrom:
        secretKeyRef:
          name: stripe-api-key
          key: api-key
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Retrieving Existing Secret Data

## Exam Tip
