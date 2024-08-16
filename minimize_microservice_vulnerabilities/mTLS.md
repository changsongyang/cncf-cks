# Mutual TLS (mTLS)

## Intropduction
- Lots of micro services
- Lots of pods communicate with each other
- Maybem attacker can listen on Pods.

mTLS means both communication fully auth each other.

Probably, seen HTTPS where server is certificate, but with mTLS both client and server have certificate, both parties have a certificate, and prevents anyone from listen in, either client or server.

mTLS is away to preventing anyone from impersonating client or server and listening on Pods to Pods traffics.

## mTLS protects against MiTM attacks
![image](https://github.com/user-attachments/assets/bb5e0abd-cb0b-476c-9dc3-58ee4353bdad)
_Source: [Kubernetes CLS Full Course Theory + Practice + Browser Scnearios [YouTube]](https://youtu.be/d9xfB5qaOfg)_


## Certificate signing

If all Pods need server and client certificates, the there will be lots of certificates with ltos of muicroservices, and we need automated solution to manage that.

Feature of Certificate signing of Kubernetes allows objtaining certificates.

1. Kubernmetes API - allows to obtain certiifcates
2. CA - Certiricates will be grnerated form central root CA.
3. Progammatic certificate access


## Processo f Signing certificates via Kubernetes API

### Requesting and Signing Certificates

1. First, requestor create CSR object to request a new certificate.
2. CSR can be approved or denied.
3. RBAC can control who can approve CSR. 


### Hands on-demo

### REauestin certificate

Creating CSR is standard process, and is not related to kubernetes.

```sh
apt-install install golang-cfssl
```


Generate CSR.

```
# CSR generated.
# ls
server.csr     # <-- CSR
server-key.pem # <-- server side key
```

###  Signing Certiicate

Frist, base64 encode the CSR file
```
$ cat server.csr | base64
LS0.....RJ
```

Create CSR object
```
$ vi csr.yaml
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

Approve CSR

```
$ kubectl certificate approve my-csr
... approved
```

```
$ kubectl get csr
NAME    ... REQUESTOR        CONDITION
my-csr  ... kubernetes-admin Approved,Issued
```

Obtaining signed certificate

```
kubectl get csr my-csr -o yaml
...
status:
  certificate: LS0...Cg== # <-- base64 encoded certificate data
```

```
kubectl get csr my-csr -o jsonpath='{.status.certificate}' | base64 -d 
```

Tips:
- Create CSR iojbect ot request a new certificate
- Management/ approve or deny request via `kubectl certificate`
- Signed certiifcate can be objtained from `status.certificate` field, and is base64 encoded
-

## Resources

- https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster
