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

### Create a policy

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

### Authorize use of policy

For a user to use a policy to validate their Pods, the user must be authorized to use the policy via RBAC.

>[!IMPORTANT]
>If a user is not authorized to use any policy, they CANNOT CREATE POD.

```yaml
apiVersion:
kind: ClusterRole
metadata:
  name: cr-use-my-psp
rules:
- apiGroup: ["policy"]
  verb: ["use"]
  resources: ["podsecuritypolicies"]
resourceNames:
- my-psp
```
