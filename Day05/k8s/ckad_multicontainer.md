# Multicontainer Patterns


- Understand multi-container pod design patterns (eg: ambassador, adaptor, sidecar)

---

## The Sidecar Pattern


- Make use of small pluggable components

--

- An example in linux is the `ps` command; displays the currently running processes but no flag to filter output

--

- use `grep`

- `ps faux | grep docker`

--

- Typically, a Pod contains a single container. However, multiple containers can be placed in the same Pod.

- Typically, a Pod contains a single container. However, multiple containers can be placed in the same Pod.

---

class: pic

## Scenario: Log-Shipping Sidecar

![sidecar](images/sidecarpattern.jpg)

---

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx

    - name: sidecar-container
      image: busybox
      command: ["sh","-c","while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
  ```

---

## The Adapter Pattern

- Why do we need an adapter container?

--

  - Establish a unified interface for our application container (main) that can be used by a third-party service .eg Prometheus 

---

class: pic

![adapter](images/adapterpattern.jpg)

---

```yaml
  containers:
  - name: webserver
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /etc/nginx/conf.d
      name: nginx-conf
      readOnly: true
  - name: adapter
    image: nginx/nginx-prometheus-exporter:0.4.2
    args: ["-nginx.scrape-uri","http://localhost/nginx_status"]
    ports:
    - containerPort: 9113
  ```

---

## The Ambassador Pattern

- Proxy connections from the application container (main) to other services

- For example, test database, staging database, production database

--

  - Which database do we connect to?

  - Use an ambassador container to proxy DB connections depending on where it runs

---

class: pic

![ambassadordb](images/ambassador-diagram.png)

---

## Scenario: Connecting to multiple Redis servers


- Another use case is when the application needs to connect to a caching server like `Redis`

- The application initiates a request to localhost:6380. As far as the application is concerned, this is the Redis server

- The Ambassador container picks up the request and relays it to the Redis servers defined in its configuration

---

class: pic

![ambassadorcache](images/ambassadorpattern.png)

---

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: ambassador-example
spec:
  containers:
  - name: redis-client
    image: redis
  - name: ambassador
    image: malexer/twemproxy
    env:
    - name: REDIS_SERVERS
      value: redis-st-0.redis-svc.default.svc.cluster.local:6379:1 redis-st-1.redis-svc.default.svc.cluster.local:6379:1
    ports:
    - containerPort: 6380
```

---

## References:



- [Kubernetes Patterns by Bilgin Ibryam and Roland Huß](https://k8spatterns.io/)

- [Kubernetes Patterns talk by Roland Huß](https://www.youtube.com/watch?v=eJmNSYvelSw)

