# Minimize use of and access to GUI elements

## Best practices
- Only expose services externaly if needed.
- Use kubectl port-forward / kubectl proxy to expose internal services/dashboards rather than via node port or LB service.

**Remember:** [Tesla Hack from 2018](https://www.wired.com/story/cryptojacking-tesla-amazon-cloud/)

# kubectl proxy

- Creates a proxy server between localhost and Kubernetes API server
- Uses connection as configured in the kubeconfig
- Allows to access API locally over http without auth


```sh
# Terminal 1
> kubectl proxy
Starting to serve on 127.0.0.1:8001

# Terminal 2
> curl localhost:8001/api/v1/componentstatuses
> curl localhost:8001/api/v1/nodes
```

# kubectl proxy-forward

- Fowards connections from a localhost-port to a pod-port
- More generic than kubectl proxy
- Can be used for all TCP traffic not just HTTP.

```sh
# Terminal 1
> kubectl run webserver --image nginx
> kubectl port-foward pod/webserver 8080:80

# Terminal 2
> curl localhost:8080 --head
```

# Deploy and Access the Kubernetes Dashboard

## Deploy

- https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

## Access

- https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md
