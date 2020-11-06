# Core concepts

- First make sure you are on v1.19, if you are not, upgrade

- Understand Kubernetes API primitives

- Create and configure basic Pods

---

## Understand Kubernetes API primitives

- basic objects: containers, pods, services, volumes, namespaces, configmaps and secrets
 
- controllers: deployments, statefulsets, daemonsets, jobs, cronjobs

- API versioning

  - Alpha: can contain bugs, disabled by default e.g v1alpha

  - Beta: code is tested and considered safe, enabled by default e.g apiv1beta

  - Stable: VX where X stands for an integer e.g v1



---

## Understand Kubernetes API primitives

- API Groups `curl -k https://<serverip>:<port>/apis command`

- `kubectl -v=8 get svc -n kube-system`


---

## Create and Configure Basic Pods

- Use `kubectl run podname --image name` > Imperative

- Use `kubectl run podname --image name --dry-run -o yaml > filename` followed by `kubectl apply -f filename` > Declarative

- List pods `kubectl get pods -o wide`

- Change image version `kubectl set image pod/nginx nginx=nginx:1.17.1`

- Use json path `kubectl get pod nginx -o jsonpath='{.spec.containers[].image}{"\n"}'`

- Inspect `kubectl logs pod` `kubectl describe pod` `kubectl exec -ti pod/nginx -- sh`

- Format the output `kubectl get pods--sort-by=.metadata.creationTimestamp`





