# Configuration

- Understand ConfigMaps

- Understand SecurityContexts

- Define an application’s resource requirements

- Create & consume Secrets

- Understand ServiceAccounts

---

## Understand ConfigMaps

- A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume

- configmap with literal values `kubectl create cm myconfigmap --from-literal=appname=myapp`

- configmap from a file `kubectl create cm keyvalcfgmap --from-file=config.txt`

- configmap from an env file `kubectl create cm envcfgmap --from-env-file=file.env`

- Mount configmaps as volumes

---

## Understand SecurityContexts

- A security context defines privilege and access control settings for a Pod or Container

- Run the a Pod/Container as whom? `runAsUser` `runAsGroup`

- Restrict Pod/Container capabilities `["NET_ADMIN", "SYS_TIME"]`

---

## Define an application’s resource requirements

- We can attach resource indications to our pods

  (or rather: to the *containers* in our pods)

- We can specify *limits* and/or *requests*

- We can specify quantities of CPU and/or memory

---

## Limits vs requests

- Limits are "hard limits" (they can't be exceeded)

  - a container exceeding its memory limit is killed

  - a container exceeding its CPU limit is throttled

- Requests are used for scheduling purposes

  - a container using *less* than what it requested will never be killed or throttled

  - the scheduler uses the requested sizes to determine placement

  - the resources requested by all pods on a node will never exceed the node size

---

## Specifying resources

- Resource requests are expressed at the *container* level

- CPU is expressed in "virtual CPUs"

  (corresponding to the virtual CPUs offered by some cloud providers)

- CPU can be expressed with a decimal value, or even a "milli" suffix

  (so 100m = 0.1)

- Memory is expressed in bytes, kilobytes, megabytes ...etc

---

## Specifying resources in practice

This is what the spec of a Pod with resources will look like:

```yaml
containers:
- name: nginx
  image: nginx
  resources:
    limits:
      memory: "100Mi"
      cpu: "100m"
    requests:
      memory: "100Mi"
      cpu: "10m"
```

This set of resources makes sure that this service won't be killed (as long as it stays below 100 MB of RAM), but allows its CPU usage to be throttled if necessary.

---

## Default values

- If we specify a limit without a request: 

  the request is set to the limit

- If we specify a request without a limit: 

  there will be no limit

  (which means that the limit will be the size of the node)

- If we don't specify anything:

  the request is zero and the limit is the size of the node

*Unless there are default values defined for our namespace!*

---

## We need default resource values

- If we do not set resource values at all:

  - the limit is "the size of the node"

  - the request is zero

- This is generally *not* what we want

  - a container without a limit can use up all the resources of a node

  - if the request is zero, the scheduler can't make a smart placement decision

- To address this, we can set default values for resources

- This is done with a LimitRange object

---

## Defining min, max, and default resources

- We can create LimitRange objects to indicate any combination of:

  - min and/or max resources allowed per pod

  - default resource *limits*

  - default resource *requests*


- LimitRange objects are namespaced

- They apply to their namespace only

---

## LimitRange example

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: my-very-detailed-limitrange
spec:
  limits:
  - type: Container
    min:
      cpu: "100m"
    max:
      cpu: "2000m"
      memory: "1Gi"
    default:
      cpu: "500m"
      memory: "250Mi"
    defaultRequest:
      cpu: "500m"
```

---

## Namespace quotas

- We can also set quotas per namespace

- Quotas apply to the total usage in a namespace

  (e.g. total CPU limits of all pods in a given namespace)

- Quotas can apply to resource limits and/or requests

  (like the CPU and memory limits that we saw earlier)

- Quotas can also apply to other resources:

  - "extended" resources (like GPUs)

  - storage size

  - number of objects (number of pods, services...)

---

## Creating a quota for a namespace

- Quotas are enforced by creating a ResourceQuota object

- ResourceQuota objects are namespaced, and apply to their namespace only

- We can have multiple ResourceQuota objects in the same namespace

- The most restrictive values are used

---

## Limiting total CPU/memory usage

- The following YAML specifies an upper bound for *limits* and *requests*:
  ```yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: a-little-bit-of-compute
    spec:
      hard:
        requests.cpu: "10"
        requests.memory: 10Gi
        limits.cpu: "20"
        limits.memory: 20Gi
  ```

These quotas will apply to the namespace where the ResourceQuota is created.

---

## Limiting number of objects

- The following YAML specifies how many objects of specific types can be created:
  ```yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: quota-for-objects
    spec:
      hard:
        pods: 100
        services: 10
        secrets: 10
        configmaps: 10
        persistentvolumeclaims: 20
        services.nodeports: 0
        services.loadbalancers: 0
        count/roles.rbac.authorization.k8s.io: 10
  ```
---

## YAML vs CLI

- Quotas can be created with a YAML definition

- ...Or with the `kubectl create quota` command

- Example:
  ```bash
  kubectl create quota my-resource-quota --hard=pods=300,limits.memory=300Gi
  ```

---

## Viewing current usage

When a ResourceQuota is created, we can see how much of it is used:

```
kubectl describe resourcequota my-resource-quota

Name:                            my-resource-quota
Namespace:                       default
Resource                         Used  Hard
--------                         ----  ----
pods                             12    100
services                         1     5
services.loadbalancers           0     0
services.nodeports               0     0
```
---

## Limiting resources in practice

- We have at least three mechanisms:

  - requests and limits per Pod

  - LimitRange per namespace

  - ResourceQuota per namespace

---

## Viewing a namespace limits and quotas

- `kubectl describe namespace` will display resource limits and quotas

.exercise[

- Try it out:
  ```bash
  kubectl describe namespace default
  ```

- View limits and quotas for *all* namespaces:
  ```bash
  kubectl describe namespace
  ```

]

---

## Create & consume Secrets

![secrets](images/secretscreate.png)

---

class: pic

![secrets](images/secretsconsume.png)

---

## Understand ServiceAccounts

- Provide an identity for pods

- When you access a cluster you're authenticated by the apiserver as a User Account

- While processes in containers inside pods are authenticated as some Service Account

- Create a service account `kubectl create serviceaccount admin`

