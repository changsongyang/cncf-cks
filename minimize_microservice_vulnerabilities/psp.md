# PodSecurityPolicy (PSP)

>[!WARNING]
>Pod security polcies has been deprecated in Kubernetes v1.21 and completely removed in v1.25.
> 
> The CKS exam is based on Kubernetes v1.30.
>
> PSP is replaced by PSA (Policy Security Admission)

## What is a PSP?
- Controls what security-related configurations pods are allowed to run in the cluster.
- Enforces desired Pod security configuration with the cluster.

Primary need for PSP is to prevent running insecure and misconfigured Pod in the cluster, as that can put cluster or other workloads at risk.

## Using Pod Security Policy

>[!IMPORTANT]
>To use PSP, first enable admission controller in kube-apiserver by updating following flag:
>
> --enable-admission-plugins=PodSecurityPolicy,...
>

>[!CAUTION]
>Pod must satisfy atleast one PSP in order to be allowed. Otherwise, no Pods will be allowed to run.
>

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: my-psp
spec:
  priviledged: false           # Can container run in priviledge mode?
  runAsUser:                   # Can container run as as any user?
    rule: RunAsAny
```

Additional, a user must be authorized to use Pod Security Policy via RBAC.

```yaml
apiVersion:
kind: ClusterRole
metadata:
  name: use-psp
rules:
- apiGroup: ["policy"]
  verb: ["use"]
  resources: ["podsecuritypolicies"]
resourceNames:
- my-psp
```
