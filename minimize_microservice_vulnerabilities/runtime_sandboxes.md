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

## Kata containers

`Kata container` uses lightweight VMs to run containers, combining the isolation of VM and speed/flexibility of containers.

## Firecracker

Same as Kata container but runs microVms, offering isolated environment for containers.
