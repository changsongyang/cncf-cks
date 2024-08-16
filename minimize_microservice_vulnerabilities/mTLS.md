# Mutual TLS (mTLS)

## Introduction
- Running microservices application in Kubernetes involves many pods communicating with each other on different nodes.
- An attacker might be able to listen in on these pod communications.

mTLS ensures that both parties in the communication are fully authenticated and traffic is encrypted. Unlike HTTPS, where only the server is authenticated with a certificate, mTLS requires both the client and the server to present certificates, preventing eavesdropping and impersonation.

## mTLS Protects Against MiTM Attacks
![image](https://github.com/user-attachments/assets/bb5e0abd-cb0b-476c-9dc3-58ee4353bdad)
_Source: [Kubernetes CLS Full Course Theory + Practice + Browser Scenarios [YouTube]](https://youtu.be/d9xfB5qaOfg)_

## Certificates Everywhere (How Can We Easily Manage Them?)
![image](https://github.com/user-attachments/assets/90a6bd7d-588e-4a22-9bac-7cbd8d1948f7)

### Solution A - ServiceMesh / Sidecar Proxy
![image](https://github.com/user-attachments/assets/48c79853-1044-4bba-9fef-b3d45a6f0155)

- A proxy sidecar container handles communication.
- The application container doesn't directly communicate with other pods; the proxy sidecar manages this.
- The proxy sidecar sends decrypted traffic to the application container.
- The application container doesn't need to change and can continue to use HTTP.
- mTLS is managed externally by a service mesh like Istio or Linkerd.

Fun fact: The proxy sidecar container is actually performing an ethical MiTM :P 

## Certificate Signing
If every pod needs both server and client certificates, the sheer number of certificates across numerous microservices requires an automated management solution. Kubernetes provides a feature for certificate signing to facilitate this.

1. Kubernetes API allows the automated acquisition of certificates.
2. Certificates are generated from a central root CA.
3. Programmatic certificate access is enabled.

## Process of Signing Certificates via Kubernetes API

### Requesting and Signing Certificates

1. The requestor creates a CSR (Certificate Signing Request) object to request a new certificate with CSR data encoded as base64.
```
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

```sh
$ kubectl create -f csr.yaml
... created
```

2. CSR is approved (RBAC permission needed to approve CSR)

```
$ kubectl certificate approve my-csr
... approved

$ kubectl get csr
NAME    ... REQUESTOR        CONDITION
my-csr  ... kubernetes-admin Approved,Issued
```

3. Obtain the certificate

```
$ kubectl get csr my-csr -o yaml
...
status:
  certificate: LS0...Cg== # <-- base64 encoded certificate data

$ kubectl get csr my-csr -o jsonpath='{.status.certificate}' | base64 -d 

```

### Hands-on Demo

#### Requesting a Certificate
Creating a CSR is a standard process and isn't specific to Kubernetes.

1. Install tool
```sh
apt-install install golang-cfssl
```

2. Generate CSR
```
# ls
server.csr     # <-- CSR
server-key.pem # <-- server-side key
```


## Resources
- https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster
