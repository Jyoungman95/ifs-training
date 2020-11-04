# Healthchecks

- Kubernetes provides two kinds of healthchecks: liveness and readiness

- Healthchecks are *probes* that apply to *containers* (not to pods)

- Each container can have two (optional) probes:

  - liveness = is this container dead or alive?

  - readiness = is this container ready to serve traffic?

- Different probes are available (HTTP, TCP, program execution)

- Let's see the difference and how to use them!

---

## Liveness probe

- Indicates if the container is dead or alive

- A dead container cannot come back to life

- If the liveness probe fails, the container is killed

  (to make really sure that it's really dead; no zombies or undeads!)

- What happens next depends on the pod's `restartPolicy`:

  - `Never`: the container is not restarted

  - `OnFailure` or `Always`: the container is restarted

---

## When to use a liveness probe

- To indicate failures that can't be recovered

  - deadlocks (causing all requests to time out)

  - internal corruption (causing all requests to error)

- Anything where our incident response would be "just restart/reboot it"

---

## Readiness probe

- Indicates if the container is ready to serve traffic

- If a container becomes "unready" it might be ready again soon

- If the readiness probe fails:

  - the container is *not* killed

  - if the pod is a member of a service, it is temporarily removed

  - it is re-added as soon as the readiness probe passes again

---

## When to use a readiness probe

- To indicate failure due to an external cause

  - database is down or unreachable

  - mandatory auth or other backend service unavailable

- To indicate temporary failure or unavailability

  - application can only service *N* parallel connections

  - runtime is busy doing garbage collection or initial data load

- For processes that take a long time to start

---

## Timing and thresholds

- Probes are executed at intervals of `periodSeconds` (default: 10)

- The timeout for a probe is set with `timeoutSeconds` (default: 1)

.warning[If a probe takes longer than that, it is considered as a FAIL]

- A probe is considered successful after `successThreshold` successes (default: 1)

- A probe can have an `initialDelaySeconds` parameter (default: 0)

- Kubernetes will wait that amount of time before running the probe for the first time

---

## Different types of probes

- HTTP request

  - specify URL of the request (and optional headers)

  - any status code between 200 and 399 indicates success

- TCP connection

  - the probe succeeds if the TCP port is open

- arbitrary exec

  - a command is executed in the container

  - exit status of zero indicates success

---

## Benefits of using probes

- Rolling updates proceed when containers are *actually ready*

  (as opposed to merely started)

- Containers in a broken state get killed and restarted

  (instead of serving errors or timeouts)

- Unavailable backends get removed from load balancer rotation

  (thus improving response times across the board)

- If a probe is not defined, it's as if there was an "always successful" probe

---

## Example: HTTP probe


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-demo
spec:
  containers:
  - name: demo
    image: demo/demo:v0.1
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 1
```

If the backend serves an error, or takes longer than 1s, 3 times in a row, it gets killed.

---

## Example: exec probe

Here is a pod template for a Redis server:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-with-liveness
spec:
  containers:
  - name: redis
    image: redis
    livenessProbe:
      exec:
        command: ["redis-cli", "ping"]
```

If the Redis process becomes unresponsive, it will be killed.

---

## Example: Readiness Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - name: app
    image: demo/app:v0.1
    ports:
    - containerPort: 3000
    readinessProbe:
      initialDelaySeconds: 2
      periodSeconds: 5
      httpGet:
        path: /ready
        port: 3000
```

---
