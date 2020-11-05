# Rolling updates

- Let's imagine you have your application deployed in Kubernetes. 

- You want to upgrade the app to version 2.0, and you wish to do so with a zero-downtime upgrade.

- The easier option is to add the newer version of the app alongside the existing ones and take out the old one.

- You will eventually migrate your production workload from version one to version two. 

- Replacing Pods one at the time is usually referred to as a rolling update.

---
class: pic

![rollout](images/rollout/01.png)

---

class: pic

![rollout](images/rollout/02.png)

---

class: pic

![rollout](images/rollout/03.png)

---

class: pic

![rollout](images/rollout/04.png)

---

class: pic

![rollout](images/rollout/05.png)

---

class: pic

![rollout](images/rollout/06.png)

---

class: pic

![rollout](images/rollout/07.png)

---

class: pic

![rollout](images/rollout/08.png)

---

class: pic

![rollout](images/rollout/09.png)

---

class: pic

![rollout](images/rollout/10.png)

---

class: pic

![rollout](images/rollout/11.png)

---

class: pic

![rollout](images/rollout/12.png)

---

class: pic

![rollout](images/rollout/13.png)

---

class: pic

![rollout](images/rollout/14.png)

---

class: pic

![rollout](images/rollout/15.png)

---

class: pic

![rollout](images/rollout/16.png)

---

class: pic

![rollout](images/rollout/17.png)

---

## This is what happens:

- Kubernetes will create a new Pod with the latest version

- Kubernetes will wait for the liveness probe to be healthy

- As soon as the readiness probe passes, the Pod is attached to the Service and is ready to receive traffic the Pod is ready

- Kubernetes will take down one of the Pods in the Deployment

- The same steps are repeated for every other Pod in the Deployment until the rollout is completed.

- Rolling updates are excellent when you wish to deliver features in production incrementally

---

# Canary Deployments

- Another option to deploy to production without disrupting live traffic is to use a Canary deployment.

- with a canary deployment you have two versions of your app (current and previous) deployed at the same time

---

class: pic

![rollout](images/rollout/18.png)

---

class: pic

![rollout](images/rollout/19.png)

---

## Using labels and selectors with Canary Deployments

- Create two Deployments: one for the existing app with Pods with a `version: 1.0.0` label and another with a `version: 2.0.0` label

- When you start, 100% of the traffic is routed to the existing application

---

```yaml

kind: Service
apiVersion: v1
metadata:
  name: canary-service
spec:
  selector:
    version: "1.0.0"
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

---

```yaml

kind: Service
apiVersion: v1
metadata:
  name: canary-service
spec:
  selector:
    component: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

---

class: pic

![rollout](images/rollout/20.png)

---

class: pic

![rollout](images/rollout/21.png)

---

```yaml

kind: Service
apiVersion: v1
metadata:
  name: canary-service
spec:
  selector:
    version: "2.0.0"
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

---

class: pic

![rollout](images/rollout/22.png)

---

# Blue-green Deployments

- Rolling updates and Canary deployments are excellent strategies to introduce incremental updates

- However, there're times when you have to introduce breaking changes to your API or application

- In that case, you can't have two versions of your application live at the same time

- A more appropriate strategy to rollout breaking changes is to use a blue-green Deployment.

---

class: pic

![rollout](images/rollout/23.png)

---

class: pic

![rollout](images/rollout/24.png)

---

class: pic

![rollout](images/rollout/25.png)

---

class: pic

![rollout](images/rollout/26.png)

- Don't forget to remove the Deployment for the previous app.

---

# Rolling back a Deployment

- Things go wrong

- When you introduce a change that breaks production, you should have a plan to roll back that change

- Kubernetes and kubectl offer a simple mechanism to roll back changes to resources such as Deployments

---

class: pic

![rollout](images/rollout/26.png)

---

class: pic

![rollout](images/rollout/27.png)

---

class: pic

![rollout](images/rollout/28.png)

---

class: pic

![rollout](images/rollout/29.png)

---

class: pic

![rollout](images/rollout/30.png)

---

class: pic

![rollout](images/rollout/31.png)

---

class: pic

![rollout](images/rollout/32.png)

---

class: pic

![rollout](images/rollout/33.png)

---


class: pic

![rollout](images/rollout/34.png)

---


class: pic

![rollout](images/rollout/35.png)

---


class: pic

![rollout](images/rollout/36.png)

---

- keeping the previous ReplicaSets around is a convenient mechanism to roll back to a previously working version of your app

- By default Kubernetes stores the last 10 ReplicaSets and lets you roll back to any of them

- But you can change how many ReplicaSets should be retained by changing the spec.revisionHistoryLimit in your Deployment

```yaml
....
spec:
  replicas: 3
  revisionHistoryLimit: 100
selector:
  matchLabels:
    name: app
    .....
```
---

.exercise[
Create a deployment for version 3.0.0

• the Deployment should have one replica

• the image should be ahmedgabercod/rollapp:3.0.0

• the Deployment should have readiness and liveness probes

• the Pods should have the following label: 
  version: 3.0.0

Can you change the service selector so that the traffic is routed only to version 2.0.0 and 3.0.0?
]

---

## Clean up

```bash
kubectl delete all --all

```
---

.exercise[
Create a deployment for version 3.0.0

• the Deployment should have one replica

• the image should be ahmedgabercod/rollapp:3.0.0

• the Deployment should have readiness and liveness probes

• the Pods should have the following label: 
  version: 3.0.0

Can you change the selector for the service so that you can route traffic to version 1.0.0 , 2.0.0 and 3.0.0 with the following split?

- 1.0.0 50%
- 2.0.0 25%
- 3.0.0 25%
]

---

## Clean up

```bash
kubectl delete all --all

```

---

.exercise[
Create a deployment for version 3.0.0

• the Deployment should have one replica

• the image should be ahmedgabercod/rollapp:3.0.0

• the Deployment should have readiness and liveness probes

• the Pods should have the following label: 
  version: 3.0.0

You should upgrade your application to version blue-green deployment.
]

---

## Clean up

```bash
kubectl delete all --all

```

---

