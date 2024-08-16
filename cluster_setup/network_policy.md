# 🔐 Network Policy

- Network policy is a firewall rule in Kubernetes.
- Network policy is implemeneted by network plugin CNI (Cilium, Calico, etc). [See full list](https://landscape.cncf.io/?view-mode=card&classify=category&sort-by=name&sort-direction=asc#runtime--cloud-native-network)
- Namespace scoped.
- Restrict `ingreses` and/or `egress` for a group of Pods based on certain rules and conditions.

## Without Network Policy
- By default, in Kubernetes every Pod can communicate with every other Pod.
- Pods are not isolated network wise.

# ✋ Hands-on

## 🎯 Scenario

#### ➕ Create 2 Pods, frontend and backend.

```sh
k run frontend --image nginx
k run backend --image nginx
```

#### ➕ Create service for frontend and backend to enable network communication.

```sh
k expose pod/frontend --port 80
k expose pod/backend --port 80
k get pod,svc
```

#### 🟢 Check network connectivity between them.

```sh
k exec frontend -- curl backend --head -s
k exec backend -- curl frontend --head -s
```


## 🛑 Default deny network policy (except DNS)

➕ Create default network policy to deny `ingress` and `egress` traffic except DNS from `default` namespace.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  - Ingress
  egress:
  - to:
    ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
```

```
k apply -f default-deny-netpol.yaml
k get netpol
```


#### 🔗 Check network conectivity from `frontend` to `backend` Pod.

```
$ k exec frontend -- curl backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:05 --:--:--     0curl: (6) Could not resolve host: b
ackend
command terminated with exit code 6

```

## 🟢 Allow `frontend` Pod to communicate with `backend` Pod

#### ➕ Create outbound (egress) network policy from `frontend` to `backend` Pod.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-f2b-egress
  namespace: default
spec:
 podSelector:
   matchLabels:
    run: frontend
 policyTypes:
 - Egress
 egress:
 - to:
   - podSelector:
      matchLabels:
       run: backend
```

```sh
k apply -f allow-f2b-egress-netpol.yaml
```

#### ➕ Create inbound (ingress) policy to `backend` from `frontend` Pod.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-b2f-ingress
  namespace: default
spec:
 podSelector:
   matchLabels:
    run: backend
 policyTypes:
 - Ingress
 ingress:
 - from:
   - podSelector:
      matchLabels:
       run: frontend
```

```sh
k apply -f allow-b2f-ingress-netpol.yaml
```

#### 🔗 Check network connectivity from `frontend` to `backend` Pod.

```sh
k exec frontend -- curl backend --head -s
```


 
