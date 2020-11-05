# Service and Selectors

--

  - ClusterIP: The default value. The service is only accessible from within the Kubernetes cluster

--

  - NodePort: This makes the service accessible on a static port on each Node in the cluster

--

  - Load Balancer: The service becomes accessible externally through a cloud provider's load balancer functionality. GCP, AWS, Azure, etc

---

class: pic

![serviceone](images/service01.png)

---

class: pic

![serviceone](images/service02.png)

---

class: pic

![serviceone](images/service03.png)

---

class: pic

![serviceone](images/service04.png)

---

class: pic

![serviceone](images/service05.png)

---


## Selector evaluation

- We can use selectors with many `kubectl` commands

- For instance, with `kubectl get`, `kubectl logs`, `kubectl delete` ... and more

.exercise[

- Get the list of pods matching selector `app=rng`:
  ```bash
  kubectl get pods -l app=rng
  kubectl get pods --selector app=rng
  ```

]

But ... why do these pods (in particular, the *new* ones) have this `app=rng` label?

---

## Where do labels come from?

- When we create a deployment with `kubectl create deployment rng`,
  <br/>this deployment gets the label `app=rng`

- The replica sets created by this deployment also get the label `app=rng`

- The pods created by these replica sets also get the label `app=rng`

- When we created the daemon set from the deployment, we re-used the same spec

- Therefore, the pods created by the daemon set get the same labels

.footnote[Note: when we use `kubectl run stuff`, the label is `run=stuff` instead.]

---

## Adding labels to pods

- We want to add the label `active=yes` to all pods that have `app=rng`

- We could edit each pod one by one with `kubectl edit` ...

- ... Or we could use `kubectl label` to label them all

- `kubectl label` can use selectors itself

.exercise[

- Add `active=yes` to all pods that have `app=rng`:
  ```bash
  kubectl label pods -l app=rng active=yes
  ```

]

---

## Labels and debugging

- When a pod is misbehaving, we can delete it: another one will be recreated

- But we can also change its labels

- It will be removed from the load balancer (it won't receive traffic anymore)

- Another pod will be recreated immediately

- But the problematic pod is still here, and we can inspect and debug it

- We can even re-add it to the rotation if necessary

  (Very useful to troubleshoot intermittent and elusive bugs)

---

## Advanced label selectors

- Service selectors are limited to a `AND`

- But in many other places in the Kubernetes API, we can use complex selectors

  (e.g. Deployment, ReplicaSet, DaemonSet, NetworkPolicy ...)

- These allow extra operations; specifically:

  - checking for presence (or absence) of a label

  - checking if a label is (or is not) in a given set

- Relevant documentation:

  [Service spec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#servicespec-v1-core),
  [LabelSelector spec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#labelselector-v1-meta),
  [label selector doc](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)

---

## Example of advanced selector

```yaml
  theSelector:
    matchLabels:
      app: portal
      component: api
    matchExpressions:
    - key: release
      operator: In
      values: [ production, preproduction ]
    - key: signed-off-by
      operator: Exists
```

This selector matches pods that meet *all* the indicated conditions.

`operator` can be `In`, `NotIn`, `Exists`, `DoesNotExist`.

A `nil` selector matches *nothing*, a `{}` selector matches *everything*.
<br/>
(Because that means "match all pods that meet at least zero condition".)

---

## Services and Endpoints

- Each Service has a corresponding Endpoints resource

  (see `kubectl get endpoints` or `kubectl get ep`)

- That Endpoints resource is used by various controllers

  (e.g. `kube-proxy` when setting up `iptables` rules for ClusterIP services)

- These Endpoints are populated (and updated) with the Service selector

- We can update the Endpoints manually, but our changes will get overwritten

- ... Except if the Service selector is empty!

---

class: pic

![serviceone](images/service01.png)

---

# Ingress

- *Services* give us a way to access a pod or a set of pods

- Services can be exposed to the outside world:

  - with type `NodePort` (on a port >30000)

  - with type `LoadBalancer` (allocating an external load balancer)

---

## Ingress resources

- Kubernetes API resource (`kubectl get ingress`/`ingresses`/`ing`)

- Designed to expose HTTP services

- Basic features:

  - load balancing
  - SSL termination
  - name-based virtual hosting

- Can also route to different services depending on:

  - URI path (e.g. `/api`→`api-service`, `/static`→`assets-service`)
  - Client headers, including cookies (for A/B testing, canary deployment...)
  - and more!

---

## Principle of operation

- Step 1: deploy an *ingress controller*

  - ingress controller = load balancer + control loop

  - the control loop watches over ingress resources, and configures the LB accordingly

- Step 2: set up DNS

  - associate DNS entries with the load balancer address

- Step 3: create *ingress resources*

  - the ingress controller picks up these resources and configures the LB

---

class: pic

![ingress](images/rollout/ingress.png)

---

class: pic

![ingress](images/rollout/nginx.png)

---

class: pic

![ingress](images/rollout/traefik.png)