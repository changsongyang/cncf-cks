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

## What is RuntimeClass?

A **RuntimeClass** is a resource that defines runtime configuration for Pods. By defining a `RuntimeClass`, you can select specific runtime features such as container runtime sandbox for certain Pods in `spec.runtimeClassName`.

```sh
> kubectl api-resources --api-group node.k8s.io
NAME             SHORTNAMES   APIVERSION       NAMESPACED   KIND
runtimeclasses                node.k8s.io/v1   false        RuntimeClass
```

### Create a RuntimeClass

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor          # ✅ `name` is the name used in Pods `spec.runtimeClassName` to apply this RuntimeClass to Pods.
handler: runsc          # ✅ `handler` referes to section in `containerd` that configures runsc.
```

### Create a Pod with `runtimeClassName: gvisor`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sandbox-pod
spec:
  runtimeClassName: gvisor
  containers:
  - name: busybox
    image: busybox
    command:
    - sh
    - -c
    - "echo Running; sleep 3600"
```

```sh
> kubectl apply -f sandbox-pod.yaml
pod/sandbox-pod created
```

### Verify the Pod is running in `gVisor` sandbox
```sh
> kubectl exec sandbox-pod -- dmesg
[   0.000000] Starting gVisor...
[   0.103288] Accelerating teletypewriter to 9600 baud...
[   0.556524] Daemonizing children...
[   1.054515] Moving files to filing cabinet...
[   1.315790] Adversarially training Redcode AI...
[   1.743080] Letting the watchdogs out...
[   2.097193] Synthesizing system calls...
[   2.230426] Forking spaghetti code...
[   2.450877] Creating cloned children...
[   2.637867] Creating process schedule...
[   3.078278] Conjuring /dev/null black hole...
[   3.259503] Setting up VFS...
[   3.478430] Setting up FUSE...
[   3.666523] Ready!
```

>[!TIP]
> Run a non-sandbox Pod and verify what do you see when you run `dmesg`?

```sh
> kubectl exec non-sandbox-pod -- dmesg | head -5
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x610f0000]
[    0.000000] Linux version 6.9.8-orbstack-00170-g7b4100b7ced4 (orbstack@builder) (clang version 17.0.6, LLD 17.0.6) #1 SMP Thu Jul 11 03:32:20 UTC 2024
[    0.000000] KASLR enabled
[    0.000000] Machine model: orbstack,virt
[    0.000000] earlycon: pl11 at MMIO32 0x0000000040001000 (options '')
```
