# Use RBAC to minimize exposure

## Introduction to RBAC

RBAC in Kubernetes is an authorization mechasim that manages permissions for users, groups within the cluster, and is configured by default in API server:

```
kube-apiserver --authorization-mode RBAC
```

RBAC allows to define roles (Roles, ClusterRoles) that specify allowed actions and then assign (RoleBinding, ClusterRoleBinding) these roles to users or groups.

>[!IMPORTANT]
> RBAC ensures that only authorized entities can perform allowed operations, enhancing security of the cluster.

### Namespaced vs Non-namespaced Resources

```
# Print namespaced resources
> kubectl api-resources --namespaced=true --api-group rbac.authorization.k8s.io
NAME           SHORTNAMES   APIVERSION                     NAMESPACED   KIND
rolebindings                rbac.authorization.k8s.io/v1   true         RoleBinding
roles                       rbac.authorization.k8s.io/v1   true         Role

# Print non-namespaced (cluster wide) resources
> kubectl api-resources --namespaced=false --api-group rbac.authorization.k8s.io
NAME                  SHORTNAMES   APIVERSION                     NAMESPACED   KIND
clusterrolebindings                rbac.authorization.k8s.io/v1   false        ClusterRoleBinding
clusterroles                       rbac.authorization.k8s.io/v1   false        ClusterRole
```

## Create scenarioes

### Scenario 1
1. Create namespace `red` and `blue`
2. User jane can only `get`, `list` secrets in namespace `red`
3. User jane can only `get`, `list` + `delete` secrets in namespace `blue`
4. Test it using `auth can-i`

<details>
<summary>Solution</summary>

```
kubectl create namespace red
kubectl create namespace blue

kubectl create role secrets-accessor --verb get,list --resource secrets --namespace red
kubectl create role secrets-accessor --verb get,list,delete --resource secrets --namespace blue


kubectl create rolebinding secrets-accessor --role secrets-accessor --user jane --namespace red
kubectl create rolebinding secrets-accessor --role secrets-accessor --user jane --namespace blue

kubectl auth can-i get secrets --as jane -n red
kubectl auth can-i list secrets --as jane -n red
kubectl auth can-i delete secrets --as jane -n red

kubectl auth can-i get secrets --as jane -n blue
kubectl auth can-i list secrets --as jane -n blue
kubectl auth can-i delete secrets --as jane -n blue
```

</details>

## Certificates and Users
