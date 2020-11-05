# Daemonsets & Statefulsets

## Daemon sets in practice

- Daemon sets are great for cluster-wide, per-node processes:

  - `kube-proxy`

  - monitoring agents

  - etc.

- They can also be restricted to run [only on some nodes](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#running-pods-on-only-some-nodes)

---

## Creating a daemon set

  ```bash
  kubectl apply -f daemon-set.yaml
  ```

--

- How do we create the YAML file for our daemon set?

  [read the docs](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#create-a-daemonset)

---

## Statefulsets

- Stateful sets are a type of resource in the Kubernetes API

  (like pods, deployments, services...)

- They offer mechanisms to deploy scaled stateful applications

- At a first glance, they look like *deployments*:

  - a stateful set defines a pod spec and a number of replicas *R*

  - it will make sure that *R* copies of the pod are running

  - that number can be changed while the stateful set is running

  - updating the pod spec will cause a rolling update to happen

- But they also have some significant differences

---

## Stateful sets unique features

- Pods in a stateful set are numbered (from 0 to *R-1*) and ordered

- They are started and updated in order (from 0 to *R-1*)

- A pod is started (or updated) only when the previous one is ready

- They are stopped in reverse order (from *R-1* to 0)

- Each pod knows its identity (i.e. which number it is in the set)

- Each pod can discover the IP address of the others easily

- The pods can persist data on attached volumes

---

## Revisiting volumes

- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/) are used for many purposes:

  - sharing data between containers in a pod

  - exposing configuration information and secrets to containers

  - accessing storage systems

- Let's see examples of the latter usage

---

## Volumes types

- There are many [types of volumes](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes) available:

  - public cloud storage (GCEPersistentDisk, AWSElasticBlockStore, AzureDisk...)

  - private cloud storage (Cinder, VsphereVolume...)

  - traditional storage systems (NFS, iSCSI, FC...)

  - distributed storage (Ceph, Glusterfs, Portworx...)

- Using a persistent volume requires:

  - creating the volume out-of-band (outside of the Kubernetes API)

  - referencing the volume in the pod description, with all its parameters

---

## Using a cloud volume

Here is a pod definition using an AWS EBS volume (that has to be created first):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-my-ebs-volume
spec:
  containers:
  - image: ...
    name: container-using-my-ebs-volume
    volumeMounts:
    - mountPath: /my-ebs
      name: my-ebs-volume
  volumes:
  - name: my-ebs-volume
    awsElasticBlockStore:
      volumeID: vol-049df61146c4d7901
      fsType: ext4
```

---

## Using an NFS volume

Here is another example using a volume on an NFS server:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-my-nfs-volume
spec:
  containers:
  - image: ...
    name: container-using-my-nfs-volume
    volumeMounts:
    - mountPath: /my-nfs
      name: my-nfs-volume
  volumes:
  - name: my-nfs-volume
    nfs:
      server: 192.168.0.55
      path: "/exports/assets"
```

---

## Individual volumes

- The Pods of a Stateful set can have individual volumes

  (i.e. in a Stateful set with 3 replicas, there will be 3 volumes)

- These volumes can be either:

  - allocated from a pool of pre-existing volumes (disks, partitions ...)

  - created dynamically using a storage system

- This introduces a bunch of new Kubernetes resource types:

  Persistent Volumes, Persistent Volume Claims, Storage Classes

  (and also `volumeClaimTemplates`, that appear within Stateful Set manifests!)

---

## Stateful set recap

- A Stateful sets manages a number of identical pods

  (like a Deployment)

- These pods are numbered, and started/upgraded/stopped in a specific order

- These pods are aware of their number

  (e.g., #0 can decide to be the primary, and #1 can be secondary)

- These pods can find the IP addresses of the other pods in the set

  (through a *headless service*)

- These pods can each have their own persistent storage

  (Deployments cannot do that)

---

## Persistent Volume Claims and Stateful sets

- A Stateful set can define one (or more) `volumeClaimTemplate`

- Each `volumeClaimTemplate` will create one Persistent Volume Claim per pod

- Each pod will therefore have its own individual volume

- These volumes are numbered (like the pods)

- Example:

  - a Stateful set is named `db`
  - it is scaled to replicas
  - it has a `volumeClaimTemplate` named `data`
  - then it will create pods `db-0`, `db-1`, `db-2`
  - these pods will have volumes named `data-db-0`, `data-db-1`, `data-db-2`

---

## What's a Storage Class?

- A Storage Class is yet another Kubernetes API resource

  (visible with e.g. `kubectl get storageclass` or `kubectl get sc`)

- It indicates which *provisioner* to use

  (which controller will create the actual volume)

- And arbitrary parameters for that provisioner

  (replication levels, type of disk ... anything relevant!)

- Storage Classes are required if we want to use [dynamic provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)

  (but we can also create volumes manually, and ignore Storage Classes)

---

## Dynamic provisioning usage

After setting up the system, all we need to do is:

*Create a Stateful Set that makes use of a `volumeClaimTemplate`.*

This will trigger the following actions.

1. The Stateful Set creates PVCs according to the `volumeClaimTemplate`.

2. The Stateful Set creates Pods using these PVCs.

3. The PVCs are automatically annotated with our Storage Class.

4. The dynamic provisioner provisions volumes and creates the corresponding PVs.

5. The PersistentVolumeClaimBinder associates the PVs and the PVCs together.

6. PVCs are now bound, the Pods can start.


