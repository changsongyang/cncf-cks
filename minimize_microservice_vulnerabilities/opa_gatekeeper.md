# OPA Gatekeeper

## What is OPA Gatekeeper?

In Kubernetes, [Admission Controllers](http://test) enforce policies on objects during create, update, delete operations. **Admission controller** is fundamental to policy enforcement in Kubernetes.

**OPA** is a general-purpose policy engine and **OPA Gatekeeper** is a CNCF graduated project that integrates OPA and Kubernetes, and adds the following on top of plain OPA:
- Kubernetes-native CRDs for extending the policy library (i.e, contraint templates)
- Kubernetes-native CRDs for instantiating the oplicy library (i.e, contraints)
- Audit functionality

By deploying OPA as an **validating admission controller** you can:

- Require container images come from pre-approved image registry.
- Require specific labels on all resources.
- Require all Pods specify resource requests and limits.
- Prevent conflicting Ingress objects from being created.

By deploying OPA as a **mutating admission controller** you can:

- Inject sidecar containers into Pods.
- Set specific annotations on all resources.
- Rewrite container images to point at pre-approved image registry.
- Include node and pod (anti-)affinity selectors on Deployments.

## Constaint Templates

A **Constaint Template** defines a CRD schema for a constraint object and the Rego logic that will enforce the constraint. A **Constraint Template** creates a new object `kind` that can be used to create **Contraint objects**.

```yaml
apiVersion: ..
kind: ConstaintTemplate
metadata:
  name: RequiredLabels
spec:
  crd:
    ... # schema
  targets:
  - target: ..
    rego: # rego logic
```


## Constraints

A "Contraint" is an instance of "Constraint Template" that attaches the logic defines in a Constraint Template to incoming objects.

```yaml
apiVersion: constraints....
kind: RequiredLabels
metadata:
  ...
spec:
  match: # which incoming object we want to apply the constraint to
    kinds:
    - apiGroup: [""]
      kinds: ["Deployment"]
    - ...
  parameters: # arguments passing to template
    labels: ["contact"]
```

## Getting hands-on

### Install OPA Gatekeeper

```sh
> helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
> helm install gatekeeper/gatekeeper --name-template=gatekeeper --namespace gatekeeper-system --create-namespace
```

```sh
> kubectl get all -n gatekeeper-system
```

### Create your first ConstraintTemplate

Source: https://github.com/open-policy-agent/gatekeeper/blob/master/demo/basic/templates/k8srequiredlabels_template.yaml
```yaml file="requiredlabels.yaml"
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: requiredlabels
spec:
  crd:
    spec:
      names:
        kind: RequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
```

```sh
> kubectl apply -f requiredlabels.yaml
constrainttemplate.templates.gatekeeper.sh/requiredlabels created
```

## Create your first Constraint

>[!IMPORTANT]
>Constraint is only enforced for incoming objects, not existing objects.

```yaml file="deployment-must-have-contact-label.yaml"
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: RequiredLabels
metadata:
  name: deploy-must-have-contact-label
spec:
  match:
    kinds:
    - apiGroups: ["apps"]
      kinds: ["Deployment"]
  parameters:
    labels: ["contact"]
```

```sh
> kubectl apply -f deployment-must-have-contact-label.yaml
requiredlabels.constraints.gatekeeper.sh/deploy-must-have-contact-label.yaml created

> kubectl get constraints
NAME                                  ENFORCEMENT-ACTION   TOTAL-VIOLATIONS
deploy-must-have-contact-label
```

## Test the constraint

```yaml file="test-deployment.yaml"
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webserver
  name: webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webserver
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webserver
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

```sh
> kubectl apply -f test-deployment.yaml
Error from server (Forbidden): error when creating "test-deployment.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [deploy-must-have-contac
t-label] you must provide labels: {"contact"}
```

```sh
> kubectl get constraint deploy-must-have-contact-label
NAME                             ENFORCEMENT-ACTION   TOTAL-VIOLATIONS
deploy-must-have-contact-label                        4

> kubectl describe constraint deploy-must-have-contact-label
```

```diff
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webserver
+   contact: dev-team
  name: webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webserver
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webserver
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

```sh
> kubectl apply -f test-deployment.yaml
deployment.apps/webserver configured
```

## Exam Tip
- For purpose of the CKS exam, you are not required to know Rego language.

## Resources
- https://www.cncf.io/blog/2023/10/09/secure-your-kubernetes-environment-with-opa-and-gatekeeper/
- https://github.com/open-policy-agent/gatekeeper/tree/master/demo
- https://github.com/snigdhasambitak/cks?tab=readme-ov-file#question-7--open-policy-agent
