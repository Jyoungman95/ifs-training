
## Init Containers

- We can define containers that should execute *before* the main ones

- They will be executed in order

  (instead of in parallel)

- They must all succeed before the main containers are started

- This is *exactly* what we need here!

- Let's see one in action

.footnote[See [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) documentation for all the details.]

---

## Defining Init Containers

.small[
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-init
spec:
  volumes:
  - name: www
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: www
      mountPath: /usr/share/nginx/html/
  initContainers:
  - name: git
    image: alpine
    command: [ "sh", "-c", "apk add git && git clone https://github.com/octocat/Spoon-Knife /www" ]
    volumeMounts:
    - name: www
      mountPath: /www/
```
]

---

## Trying the init container

.exercise[

- Create the pod:
  ```bash
  kubectl create -f nginx-4-with-init.yaml
  ```

- Try to send HTTP requests as soon as the pod comes up

<!--
```key ^D```
```key ^C```
-->

]

- This time, instead of "403 Forbidden" we get a "connection refused"

- NGINX doesn't start until the git container has done its job

- We never get inconsistent results

  (a "half-ready" container)

---

## Other uses of init containers

- Load content

- Generate configuration (or certificates)

- Database migrations

- Waiting for other services to be up

  (to avoid flurry of connection errors in main container)

- etc.

---