# Container Runtime Sandboxes

## What is container runtime?

**A container runtime** is responsible for **running containers on a host system**.
- Docker Enginer
- containerd

## What is container runtime sandboxes?

**A container runtime sandboxes** is responsible for running container on a host system **with additional isolation and security features.**
- gVisor/runsc - Developed by Google.
- Kata containers
- Firecrackers - Developed by AWS.

## Use cases
- Running untrusted workloads
- Multi-tenant environments 

>[!IMPORTANT]
>The extra security of container runtime sandboxes comes with performance trade off.

## gVisor/runsc

`gVisor/runsc` provides isolation by intercepting system calls made by containers and implements them in user-space, reducuing the reliance on host's kernel.

<img src='https://github.com/user-attachments/assets/a211e1d5-17a5-45ae-a329-462771881b79' height=250 />

## Kata containers

`Kata container` uses lightweight VMs to run containers, combining the isolation of VM and speed/flexibility of containers.

## Firecracker

Same as Kata container but runs microVms, offering isolated environment for containers.

# Hands on

## Building a Sandbox

### ✅ Install gVisor runtime on control plane + worker nodes

See https://gvisor.dev/docs/user_guide/install/

### ✅ Configure containerd to interact with runsc

See https://gvisor.dev/docs/user_guide/containerd/quick_start/

### ✅ Setup a RuntimeClass to designate which Pod need to run sandboxed runtime

## What is RuntimeClass?

A **RuntimeClass** is a resource that defines runtime configuration for Pods. By defining a `RuntimeClass`, you can select specific runtime features such as container runtime sandbox for certain Pods in `spec.runtimeClassName`.

```sh
> kubectl api-resources --api-group node.k8s.io
NAME             SHORTNAMES   APIVERSION       NAMESPACED   KIND
runtimeclasses                node.k8s.io/v1   false        RuntimeClass
```

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: example-runtimeclass          # ✅ `name` is the name used in Pods `spec.runtimeClassName` to apply this RuntimeClass to Pods.
handler: example-handler              # ✅ `handler` referes to section in `containerd` that configures runsc.
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  runtimeClassName: example-runtimeclass
  container:
  - name:
    image: busybox
```
