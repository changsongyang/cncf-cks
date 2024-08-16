# Network Security Policy

# What is Network Policy?

- Network policy is a firewall rule in Kubernetes
- Network policy is implemeneted by network plugin CNI (Cilium, Calico, etc). [See full list](https://landscape.cncf.io/?view-mode=card&classify=category&sort-by=name&sort-direction=asc#runtime--cloud-native-network)
- Namespace level
- Restrict `ingreses` and/or `egress` for a group of Pods based on certain rules and conditions.

## Without Network Policy
- By default, in Kubernetes every Pod can communicate with every other Pod.
- Pods are not isolated network wise.

# Hands-on

## Deny all outgoing traffic

Denies all outgoing traffic **from** Pods with label **id=frontend** in namespace default.
 
```yaml
apiVersion: v1
kind: NetworkPolicy
metadata:
  name: deny-all-egress
  namespace: default
spec:
  podSelector:
    matchLabels:
      id: frontend
  policyTypes:
  - Egress
```

