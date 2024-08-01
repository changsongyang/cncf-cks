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

## Retrieving existing Secret data via kubectl

```
> kubectl get secret stripe-api-key -o yaml
apiVersion: v1
data:
 api-key: YWJjMTIzZGVmNDU2
kind: Secret
metadata:
  creationTimestamp: "2024-07-28T08:14:44Z"
  name: stripe-api-key
  namespace: default
  resourceVersion: "4781"
  uid: 2a8a7892-39d1-4ba0-bfb2-7199744a528d
type: Opaque

> kubectl get secret stripe-api-key -o yaml | yq '.data.api-key | @base64d'
abc123def456
```

## Retrieving existing Secret data via crictl

```sh
> crictl inspect POD_ID
> cat /proc/POD_PROCESS_ID/root/...
```

## Encrypt Secrets in ETCD at rest

- By default, the API server stores plain-text representations of resources into etcd, with no at-rest encryption.

>[!Tip]
>If `kube-apiserver` is running without `--encryption-provider-config` flag then you do not have encryption at rest is enabled.
>```
>$ grep encryption-provider-config /etc/kubernetes/manifests/kube-apiserver.yaml || echo NO;
>```

>[!Tip]
>If `kube-apiserver` is running with `--encryption-provider-config` flag and configuration file have `identity` provider as first encyption provider then you do not have encryption at rest is enabled.

### Available providers

[See here](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#providers)

### Key storage
- Local key storage
- Managed (KMS) key storage

### Generate the encryption key

```sh
> head -c 32 /dev/urandom | base64
wxbsMJqCCfuyu6tP3fCgdLvL4zZ9fttl+6pghk1wvRM=
```
**NOTE:** Replicate the encryption key to every other control plane node. At a minimum, use encyption in trasit, e.g: secure shell (SSH).

### Write encryption configuration file

```yaml
---
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
      - pandas.awesome.bears.example
    providers:
      - aescbc:
          keys:
            - name: key1
              # See the following text for more details about the secret value
              secret: <BASE 64 ENCODED SECRET>
      - identity: {} # this fallback allows reading unencrypted secrets;
                     # for example, during initial migration
```

### Use the encyption configuration file

First, we need to mount the encyption config file `kube-apiserver` static pod.

```sh
> touch /etc/kubernetes/enc/enc.yaml
> vim /etc/kubernetes/manifests/kube-apiserver.yaml
- encyption-provider-config=/etc/kubernetes/enc/enc.yaml
volumeMounts:
- name: enc
  mountPath: /etc/kubernetes/enc
  readOnly true
volumes:
- name: enc
  hostPath:
    path: /etc/kubernetes/enc
    type: DirectoryOrCreate
```

### Verify newly written data is encrypted

After restarting `kube-apiserver`, any newly created or updated Secret (or any other resource kinds configured in `EncryptionConfiguration`) should be encrypted when written to etcd.

```sh
> grep etcd /etc/kubernetes/manifest/kube-apiserver.yaml

> ETCDCTL_API=3 etcdctl get /registry/secrets/default/stripe-api-key [...] | hexdump -C
# Secret is prefix with k8s:enc:aescbc:v1:key1
# [...] must be additional arguments for connect etcd, e.g: --cacert, --cert, --key, etc.
```

### Verify Secret is decrypted when retrieved via API
```sh
> kubectl get secrets stripe-api-key -o yaml -n default | yq '.data.api-key | @base64d'
```

### Encrypt already stored secrets

The command below reads all Secrets and then updates them with the same data, 

```sh
> kubectl get secrets -A -o json | kubectl replace -f -
```

Once all Secrets are encrypted, you can remove the `identity` part of the encryption configuration.

```diff
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <BASE 64 ENCODED SECRET>
-      - identity: {} # REMOVE THIS LINE
```

## Decrypt all data

To disable encryption at rest, place the identity provider as the first entry in your encryption configuration file.

```diff
---
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      # list any other resources here that you previously were
      # encrypting at rest
    providers:
      - identity: {} # add this line
      - aescbc:
          keys:
            - name: key1
              secret: <BASE 64 ENCODED SECRET> # keep this in place make sure it comes after "identity"
```

```sh
> kubectl get secrets -A -o json | kubectl replace -f -
```

## References
- https://kubernetes.io/docs/concepts/configuration/secret/
- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
