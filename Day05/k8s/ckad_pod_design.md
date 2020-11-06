# Pod Design

  - Understand how to use Labels, Selectors and Annotations
  
  - Understand Deployments and how to perform rolling updates
  
  - Understand Deployments and how to perform rollbacks
  
  - Understand Jobs and CronJobs

---

## Understand how to use Labels, Selectors and Annotations

- List pods and show their labels `kubectl get pods --show-labels`

- List pods with specific key and value `kubectl get pods -l env=dev`

  - `kubectl get pods -l 'env in (dev,prod)' --show-labels`

- List pods with specific key `kubectl get pods -L env`

- Set label `kubectl label pod/nginx key=value`

- Remove label `kubectl label pod/nginx key-`

- Annotate the pods `kubectl annotate pod key=value`

- Remove annotation `kubectl annotate pod key-`

---

## Understand Deployments and how to perform rolling updates

- Create deployment `kubectl create deployment webapp --image=nginx --dry-run -o yaml > webapp.yml`

- Delete deployment `kubectl delete deployment webapp`

- Scale deployment `kubectl scale deployment webapp --replicas 5`

- Check deployment rollout status `kubectl rollout status deployment webapp`

- Check deployment rollout history `kubectl rollout history deployment webapp`

- Rollout to specific version `kubectl rollout undo deployment webapp --to-revision=2`

---

## Understand Deployments and how to perform rollbacks

- Rollback a deployment `kubectl rollout undo deployment webapp`

- Rollout to specific version `kubectl rollout undo deployment webapp --to-revision=2`

---

## Understand Jobs and CronJobs

- Create a Job `kubectl create job hello-job --image busybox --dry-run -o yaml -- echo "Hello" > hello-job.yaml`

- Create a Cronjob 

  `kubectl create cronjob date-job --image=busybox --schedule="*/1 * * * *" -- bin/sh -c "date; echo Hello from kubernetes cluster"`

- activeDeadlineSeconds: automatically terminates job after a period in seconds

- parallelism: maximum number of pods running at any given time

- completions: number of times the jobs can run

- backoffLimits: number of retries before making the job failed

