# Ingress objects with security controls

## What is Ingress?

Ingress exposes HTTP nad HTTPS routes from outside the cluster to services within the clusters, plus offers:
- Name-based virtual hosting
- SSL/TLS termination
- Traffic load-balancing

### How does it work?

An [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) is responsbility for reconciling the Ingress object, usually with Ingress-managed external load-balancer, e.g: [ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/)

>[!WARNING]
> If there's no ingress controller running in the cluster, then ingress resource won't work.

### Deploy an ingress-nginx controller

- https://kubernetes.github.io/ingress-nginx/deploy/#quick-start
```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```


# Setup an Ingress (HTTP) with services

### Create deployment and service
```
kubectl create deployment app --image=httpd --port=80

kuectl expose deployment app
```

### Create an ingress resource
```
kubectl create ingress app-ingress --class=nginx --rule="app.me/*=app:80"
```

### Access the service via ingress
```
# Ingress HTTP node port (30959), e.g: 80:30959/TCP
kubectl get svc ingress-nginx-controller -n ingress-nginx

curl http://app.me:30959 --resolve app.me:30959:127.0.0.1
```

# Secure an Ingress with TLS

### Requirements

- Host a website in Kubernetes cluster
- The website should be reachable on mywebsite.com and protected with HTTPS.

### Create self-signed certificate and secret

```
openssl req -x509 -out tls.pem -keyout tls.key -newkey rsa:4096 -days 365 -nodes
# Common Name (e.g. server FQDN or YOUR name) []: mywebsite.com

kubectl create secret tls mywebsite-tls --cert tls.pem --key tls.key
```

### Deploy web application

```
kubectl create deployment mywebsite --image httpd --replicas 2
kubectl expose deployment mywebsite --port 80

# Test
kubectl run test --image nginx
kubectl exec -it test -- curl mywebsite.default.svc.cluster.local:80 -s
kubectl delete pod test --force
```


### Create an ingress with TLS rule
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mywebsite-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - mywebsite.com
    secretName: mywebsite-tls 
  rules:
  - host: mywebsite.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mywebsite
            port:
              number: 80

```

```
kubectl apply -f mywebsite-ingress.yaml
kubectl get ingress mywebsite-ingress
```


### Access via curl

```
curl https://mywebsite.com:31368 --resolve mywebsite.com:31368:127.0.0.1 -kv
```
