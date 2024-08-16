# Mutual TLS (mTLS)

## Introduction

mTLS means both communication fully auth each other.

Probably, seen HTTPS where server have certificate, but with mTLS both both parties (client & serveR) have a certificate, which prevents anyone from listen in, either client or server.

mTLS is away to preventing anyone from impersonating client or server and listening on Pod to Pod network traffic.

## Certificate signing

If all Pods needs server and client certificates, then there will be lots of certificates to manage depeneding on size of microservice application.

Kubernetes API allows signing and obtaining certificates via CSR resource.

## Process of Signing certificates via Kubernetes API

1. Requestor creates CSR object to request a new certificate.
2. Signer approve or deny CSR request via `kubectl certificate approve/deny`.
3. Requestor obtains the signed certificate from CSR object `status.certificate` field, certificate is base64 encoded.

### Hands on-demo

#### 1. Creating CSR is standard process, and is not related to kubernetes.

```sh
apt-install install golang-cfssl

# CSR generated.
# ls
server.csr     # <-- CSR
server-key.pem # <-- server side key
```

#### 1.1. Create CSR object with base64 encoded data of above CSR

```
$ cat server.csr | base64
LS0.....RJ
```

```yaml
apiVersion: ..
kind: CertificateSigningRequest
metadata:
  name: my-csr
spec:
  request: | # <-- multi-line value
    # <base74-encoded-csr>
    LS0tL
    ...
    ...
    LS0tLQo=
  signerName: kubernetes.io/kubelet-serving
  usages:
  - digital signature
  - key encipherment
  - server auth
```

```
kubectl create 0f csr.yaml
... created
```

```
$ kubectl get csr
NAME    ... REQUESTOR        CONDITION
my-csr  ... kubernetes-admin Pending
```

#### 2. Approve CSR request

```
$ kubectl certificate approve my-csr
... approved
```

```
$ kubectl get csr
NAME    ... REQUESTOR        CONDITION
my-csr  ... kubernetes-admin Approved,Issued
```

#### 3. Obtaining signed certificate

```
kubectl get csr my-csr -o yaml
...
status:
  certificate: LS0...Cg== # <-- base64 encoded certificate data
```

```
kubectl get csr my-csr -o jsonpath='{.status.certificate}' | base64 -d 
```

## Resources

- https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster
