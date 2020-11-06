## Network policies

- Namespaces help us to *organize* resources

- Namespaces do not provide isolation

- By default, every pod can contact every other pod

- By default, every service accepts traffic from anyone

- If we want this to be different, we need *network policies*

---

## What's a network policy?

A network policy is defined by the following things.

- A *pod selector* indicating which pods it applies to

  e.g.: "all pods in namespace `blue` with the label `zone=internal`"

- A list of *ingress rules* indicating which inbound traffic is allowed

  e.g.: "TCP connections to ports 8000 and 8080 coming from pods with label `zone=dmz`,
  and from the external subnet 4.42.6.0/24, except 4.42.6.5"

- A list of *egress rules* indicating which outbound traffic is allowed

A network policy can provide ingress rules, egress rules, or both.

---

## How do network policies apply?

- A pod can be "selected" by any number of network policies

- If a pod isn't selected by any network policy, then its traffic is unrestricted

  (In other words: in the absence of network policies, all traffic is allowed)

- If a pod is selected by at least one network policy, then all traffic is blocked ...

  ... unless it is explicitly allowed by one of these network policies

---

class: extra-details

## Traffic filtering is flow-oriented

- Network policies deal with *connections*, not individual packets

- Example: to allow HTTP (80/tcp) connections to pod A, you only need an ingress rule

  (You do not need a matching egress rule to allow response traffic to go through)

- This also applies for UDP traffic

  (Allowing DNS traffic can be done with a single rule)

- Network policy implementations use stateful connection tracking

---

## The rationale for network policies

- In network security, it is generally considered better to "deny all, then allow selectively"

  (The other approach, "allow all, then block selectively" makes it too easy to leave holes)

- As soon as one network policy selects a pod, the pod enters this "deny all" logic

- Further network policies can open additional access

- Good network policies should be scoped as precisely as possible

- In particular: make sure that the selector is not too broad

  (Otherwise, you end up affecting pods that were otherwise well secured)

---

## Our first network policy

This is our game plan:

- run a web server in a pod

- create a network policy to block all access to the web server

- create another network policy to allow access only from specific pods

---

## Running our test web server

.exercise[

- Let's use the `nginx` image:
  ```bash
  kubectl create deployment testweb --image=nginx
  ```

<!--
```bash
kubectl wait deployment testweb --for condition=available
```
-->

- Find out the IP address of the pod with one of these two commands:
  ```bash
  kubectl get pods -o wide -l app=testweb
  IP=$(kubectl get pods -l app=testweb -o json | jq -r .items[0].status.podIP)
  ```

- Check that we can connect to the server:
  ```bash
  curl $IP
  ```
]

The `curl` command should show us the "Welcome to nginx!" page.

---

## Looking at the network policy

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all-for-testweb
spec:
  podSelector:
    matchLabels:
      app: testweb
  ingress: []
```

---

## Allowing connections only from specific pods

- We want to allow traffic from pods with the label `run=testcurl`

- This label is automatically applied when we do `kubectl run testcurl ...`

.exercise[

- Apply another policy:
  ```yaml
  kind: NetworkPolicy
  apiVersion: networking.k8s.io/v1
  metadata:
    name: allow-testcurl-for-testweb
  spec:
    podSelector:
      matchLabels:
        app: testweb
    ingress:
    - from:
      - podSelector:
          matchLabels:
            run: testcurl
  ```

]

---

## Testing the network policy

- Let's create pods with, and without, the required label

.exercise[

- Try to connect to testweb from a pod with the `run=testcurl` label:
  ```bash
  kubectl run testcurl --rm -i --image=ahmedgabercod/net -- curl -m3 $IP
  ```

- Try to connect to testweb with a different label:
  ```bash
  kubectl run testkurl --rm -i --image=ahmedgabercod/net -- curl -m3 $IP
  ```

]

The first command will work (and show the "Welcome to nginx!" page).

The second command will fail and time out after 3 seconds.

(The timeout is obtained with the `-m3` option.)

---

## An important warning

- Some network plugins only have partial support for network policies


---

## Network policies, pods, and services

- Network policies apply to *pods*

- A *service* can select multiple pods

  (And load balance traffic across them)

- It is possible that we can connect to some pods, but not some others

  (Because of how network policies have been defined for these pods)

- In that case, connections to the service will randomly pass or fail

  (Depending on whether the connection was sent to a pod that we have access to or not)

---

## Network policies and namespaces

- A good strategy is to isolate a namespace, so that:

  - all the pods in the namespace can communicate together

  - other namespaces cannot access the pods

  - external access has to be enabled explicitly

---

## Cleaning up our network policies

.exercise[

- Remove all network policies:
  ```bash
  kubectl delete networkpolicies --all
  ```

]