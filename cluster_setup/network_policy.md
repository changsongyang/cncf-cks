# 🔐 Network Policy

- Network policy is a firewall rule in Kubernetes that describes what network traffic is allowed for a set of Pods.
- Network policy is implemeneted by network plugin CNI (Cilium, Calico, etc). [See full list](https://landscape.cncf.io/?view-mode=card&classify=category&sort-by=name&sort-direction=asc#runtime--cloud-native-network)
- Namespace scoped.
- Restrict `ingreses` and/or `egress` for a group of Pods based on certain rules and conditions.

## Without Network Policy
- By default, in Kubernetes every Pod can communicate with every other Pod.
- Pods are not isolated network wise.

## Network Policy Spec

```sh
kubectl explain networkpolicy.spec
```

# ✋ Hands-on

## 🎯 Scenario

#### ➕ Create 2 Pods, frontend and backend in default namespace.

```sh
k run frontend --image nginx
k run backend --image nginx
```

#### ➕ Create 1 Pod, database in db namespace.

```sh
k create ns db
k label ns/db ns=db
k run database --image nginx -n db
```

#### ➕ Create service for frontend, backend and database Pod to enable network communication.

```sh
k expose pod/frontend --port 80
k expose pod/backend --port 80
k expose pod/database --port 80 -n db
k get pod,svc
k get pod,svc -n db
```

#### 🟢 Check network connectivity between them.

```sh
k exec frontend -- curl backend
k exec backend -- curl frontend
k exec backend -- curl database
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

```sh
k apply -f default-deny-netpol.yaml
k get netpol
```


#### 🔗 Check network conectivity from `frontend` to `backend` Pod.

```sh
> k exec frontend -- curl backend

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
> k exec frontend -- curl backend --head -s
HTTP/1.1 200 OK
Server: nginx/1.27.1
Date: Fri, 16 Aug 2024 19:30:03 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 12 Aug 2024 14:21:01 GMT
Connection: keep-alive
ETag: "66ba1a4d-267"
Accept-Ranges: bytes
```


## 🟢 Allow `backend` Pod to communicate with `database` Pod across namespace

#### ➕ Create outbound (egress) network policy from `backend` to `database` Pod.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-b2d-egress
  namespace: default
spec:
 podSelector:
   matchLabels:
    run: backend
 policyTypes:
 - Egress
 egress:
 - to:
   - namespaceSelector:
      matchLabels:
       ns: db
```

```sh
k apply -f allow-b2d-egress-netpol.yaml
```

#### 🔗 Check network connectivity from `backend` to `database` Pod across namespace.

```sh
> k exec backend -- curl --head -s database.db
HTTP/1.1 200 OK
Server: nginx/1.27.1
Date: Fri, 16 Aug 2024 20:02:00 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 12 Aug 2024 14:21:01 GMT
Connection: keep-alive
ETag: "66ba1a4d-267"
Accept-Ranges: bytes
```

### 🛠️ Try it yourself
- Put default deny network policy in `db` namespace, and see how you can get `backend` Pod in `default` namespace to communicate with `database` Pod in `db` namespace.

