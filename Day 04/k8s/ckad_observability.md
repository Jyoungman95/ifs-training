# Observability

- Understand LivenessProbes and ReadinessProbes

- Understand container logging

- Understand how to monitor applications in Kubernetes

- Understand debugging in Kubernetes

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

.warning[**Do not** use liveness probes for problems that can't be fixed by a restart]

- Otherwise we just restart our pods for no reason, creating useless load

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

  (more on that later)

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

## Understand logging, monitoring, and debugging in Kubernetes

- `kubectl logs ${POD_NAME}`

- `kubectl describe pod ${POD_NAME}`

- `kubectl get events`

- `kubectl exec -ti pod ${POD_NAME} -- bash` then inspect /var/log/*.log

