# Security context

## What is a security context?

A **security context** defines privileges and access control settings for a Pod or Container.

## Configuring a security context

```diff
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
# Pod-level: Applies to all containers in this Pod.
+ securityContext:
+   runAsUser: 1000
+   # more settings as key:value pair
  containers:
  - name: nginx
    image: nginx
# Container-level: Only applies to this container (pod-level override)
+ securityContext:
+   runAsUser: 2000
+   # more settings as key:value pair
```

> [!NOTE]
> `Pod-level` and `Container-level` security context have common settings (e.g: `runAsUser`) but they can also have different settings that only applies to that specific level.

For a full list of available security context settings, see the Kubernetes API reference:
- [PodSecurityContext v1 core](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#podsecuritycontext-v1-core)


## Gettings hands-on

### Create a pod to playaround

```sh
$ kubectl run sc-pod --image busybox \
   --dry-run=client -o yaml --command -- sh -c "sleep 3600" | tee sc-pod.yaml
```

_File: sc-pod.yaml_
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 3600
    image: busybox
    name: sc-pod
```

```sh
$ kubectl apply -f sc-pod.yaml
$ kubectl get pods -l run=sc-pod -w
```

### By default, container runs as user set in container image

>[!WARNING]
> Recommend: Always set user/group id greater than 0 (root UID is 0)

```sh
$ kubectl exec sc-pod -- id
uid=0(root) gid=0(root) groups=0(root),10(wheel)
```

### Avoid running as root even if container image sets root user

```diff
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 3600
    image: busybox
    name: sc-pod
+   securityContext:
+     runAsNonRoot: true
```

```sh
$ kubectl replace -f sc-pod.yaml --force
pod "sc-pod" deleted
pod/sc-pod replaced

$ kubectl describe pod -l run=sc-pod | grep -i error
      Reason:       CreateContainerConfigError
  Warning  Failed   17s (x3 over 34s)  kubelet  Error: container has runAsNonRoot and image will run as root (pod: "sc-pod_default(7d407586-3695-4f30-979e
-ad884f548b19)", container: sc-pod)
```

### Run a container as non-root (i.e, uid > 0)
```diff
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 3600
    image: busybox
    name: sc-pod
+   securityContext:
+     runAsUser: 5000
```

```sh
$ kubectl replace -f sc-pod.yaml --force
pod "sc-pod" deleted
pod/sc-pod replaced

$ kubectl exec sc-pod -- id
uid=5000 gid=0(root) groups=0(root)
```

> [!TIP]
> Can you try to configure the security context so container run as group `5000`?

### Privilege escalation

`allowPrivilegeEscalation` setting controls wheather container process can gain more privilege than it's parent process.

Is always `true` when container:
- is run as privilege, or has `CAP_SYS_ADMIN`

>[!WARNING]
> Recommend: Always set it to `false` for best container security practice.

```diff
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  securityContext:
    runAsUser: 5000
  containers:
  - command:
    - sh
    - -c
    - sleep 3600
    image: busybox
    name: sc-pod
+   securityContext:
+     allowPrivilegeEscalation: false
```

```diff
$ kubectl replace -f sc-pod.yaml --force
pod "sc-pod" deleted
pod/sc-pod replaced

# On the node where container is running
$ kubectl get pods -l run=sc-pod -o wide
$ CONTAINER_ID=$(crictl ps --name sc-pod -q)
$ crictl inspect $CONTAINER_ID | grep no_new_privs
+         "no_new_privs": true,
```

### Privileged container

A privileged container is a container that runs as root user, and is directly mapped to host machine root user.

>[!Note]
>By default, containers do not as priviledge mode.

>[!CAUTION]
>Be careful when running container as privileged, as it can compromise the cluster.

```diff
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 3600
    image: busybox
    name: sc-pod
+   securityContext:
+     privileged: true
```

```diff
$ kubectl replace -f sc-pod.yaml --force
pod "sc-pod" deleted
pod/sc-pod replaced

# On the node where container is running
$ kubectl get pods -l run=sc-pod -o wide
$ CONTAINER_ID=$(crictl ps --name sc-pod -q)
$ crictl inspect $CONTAINER_ID | grep privileged
+         "privileged": true,
```


## Additional resources
- [Kubernetes Documentation: Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
