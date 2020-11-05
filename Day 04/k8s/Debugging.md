# Debugging

- Things go wrong

--

  - Cluster components: API server, Controller manager, Scheduler, and ETCD 

--

  - Node health 

--

  - Application 

--

  - Pod failure 

--

  - Services running

---

## Application Introspection and Debugging

- Once your application is running, you'll inevitably need to debug problems with it. 

- Earlier we described how you can use kubectl get pods to retrieve simple status information about your pods. 

- Using `kubectl describe pod` to fetch details about pods

- Check Events in running Pods

---
## Example: debugging Pending Pods

- Check resource request limits 

- Use `kubectl get events` 

```bash
  fit failure on node (kubernetes-node-6ta5): Node didn't have enough resource: CPU, requested: 1000, used: 1420, capacity: 2000
  fit failure on node (kubernetes-node-wul5): Node didn't have enough resource: CPU, requested: 1000, used: 1100, capacity: 2000
  ```

---

## Debugging Pods

- The first step in debugging a pod is taking a look at it 
  
  `kubectl describe pods ${POD_NAME}`

  `kubectl get events`

  `kubectl get nodes`

  `kubectl get pods -o wide`

  `kubectl logs ${POD_NAME}`

- Check Status: Pending, CrashLoopBackOff, ErrImagePull, Container Creating

---

## CrashLoopBackOff

- 99% of the times indicates an application problem

- try and run the container locally first

- check application logs `kubectl logs ${POD_NAME}`

---

## ErrImagePull

- Check image tags

- Check ImagePullPolicy; the latest tag sets the imagePullPolicy to Always implicitly

- Edit the Pod and change to IfNotPresent explicitly

- Check Container Runtime logs (docker)

---

## Container Creating

- Check secrets and configmaps

- Check if image is pulled from private registry

---

## Customizing the termination message

- Kubernetes retrieves termination messages from the termination message file specified in the terminationMessagePath field of a Container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: msg-path-demo
spec:
  containers:
  - name: msg-path-demo-container
    image: debian
    terminationMessagePath: "/tmp/my-log"
```

---

## Debugging Services

- Service is not working properly

- Does the Service exist?

- Does the Service work by DNS name? `nslookup` 

- Does the Service work by IP? `ping` `curl` `wget -qO- 10.0.1.175:80`

- Is the Service defined correctly?

- Does the Service have any Endpoints? `kubectl get endpoints`

---

## Summary

- Check Pod/Deployment Status, Events and logs

- Check Service endpoints, labels, ports, and type


[Reference](https://kubernetes.io/docs/tasks/debug-application-cluster/)
