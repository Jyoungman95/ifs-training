# State Persistence

- Understand PersistentVolumeClaims for storage

---

## Understand PersistentVolumeClaims for storage

- Our Pods can use a special volume type: a *Persistent Volume Claim*

- A Persistent Volume Claim (PVC) is also a Kubernetes resource

  (visible with `kubectl get persistentvolumeclaims` or `kubectl get pvc`)

- A PVC is not a volume; it is a *request for a volume*

- It should indicate at least:

  - the size of the volume (e.g. "5 GiB")

  - the access mode (e.g. "read-write by a single pod")

---

## What's in a PVC?

- A PVC contains at least:

  - a list of *access modes* (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)

  - a size (interpreted as the minimal storage space needed)

- It can also contain optional elements:

  - a selector (to restrict which actual volumes it can use)

  - a *storage class* (used by dynamic provisioning, more on that later)

---

## What does a PVC look like?

Here is a manifest for a basic PVC:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
   name: my-claim
spec:
   accessModes:
     - ReadWriteOnce
   resources:
     requests:
       storage: 1Gi
```
---

## Using a Persistent Volume Claim

Here is a Pod definition like the ones shown earlier, but using a PVC:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-a-claim
spec:
  containers:
  - image: ...
    name: container-using-a-claim
    volumeMounts:
    - mountPath: /my-vol
      name: my-volume
  volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName: my-claim
```

---

## Creating and using Persistent Volume Claims

- PVCs can be created manually and used explicitly

  (as shown on the previous slides)

- They can also be created and used through Stateful Sets

  (this will be shown later)

---

## Lifecycle of Persistent Volume Claims

- When a PVC is created, it starts existing in "Unbound" state

  (without an associated volume)

- A Pod referencing an unbound PVC will not start

  (the scheduler will wait until the PVC is bound to place it)

- A special controller continuously monitors PVCs to associate them with PVs

- If no PV is available, one must be created:

  - manually (by operator intervention)

  - using a *dynamic provisioner* (more on that later)
