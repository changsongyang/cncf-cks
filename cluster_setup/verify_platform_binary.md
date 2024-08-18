# Verify platform binaries

## What is Hash?

A hash is a unique fingerprint created from a file to verify its authenticity.

Hashing is a one-way process that generates a hash from a file, but you cannot reverse the process to retrieve the original file. Common hashing algorithms include SHA and MD5.

## Download and verify binaries

1. Download Kubernetes release from GitHub

```
wget https://dl.k8s.io/v1.31.0/kubernetes-server-linux-amd64.tar.gz
```


2. Generate and verify the hash of binaries

```
# SHA512 Hash:
# 4d73777e4f139c67c4551c1ca30aefa4782b2d9f3e5c48b8b010ffc329065e90ae9df3fd515cc13534c586f6edd58c3324943ce9ac48e60bb4fa49113a2e09d4
sha512sum kubernetes-server-linux-amd64.tar.gz
```


## Verify binaries from container


### Compare API server binary running inside container

1. Check running API server version

```
kubectl -n kube-system get pods $POD_NAME -o yaml | grep image
```

2. Download the binaries from official source and generate hash

```
K8S_VERSION=1.31.0
wget https://dl.k8s.io/v${K8S_VERSION}/kubernetes-server-linux-amd64.tar.gz

tar zxf kubernetes-server-linux-amd64.tar.gz
sha512sum kubernetes/server/bin/kube-apiserver | tee compare
```


3. Verify the API server binary running on the cluster

```
crictl ps | grep kube-apiserver
ps auxf | grep kube-apiserver
find /proc/$PID/root | grep kube-apiserver
sha512sum /proc/$PID/root/usr/local/bin/kube-apiserver | tee -a compare
```

4. Compare the hashes match

```
cat compare | uniq
```
