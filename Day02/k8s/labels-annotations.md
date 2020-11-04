# Labels and Annotations

- Most Kubernetes resources can have *labels* and *annotations*

- Both labels and annotations are arbitrary strings

  (with some limitations that we'll explain in a minute)

- Both labels and annotations can be added, removed, changed, dynamically

- This can be done with:

  - the `kubectl edit` command

  - the `kubectl label` and `kubectl annotate`

  - ... many other ways! (`kubectl apply -f`, `kubectl patch`, ...)

---

## Viewing labels and annotations

- Let's see what we get when we create a Deployment

.exercise[

- Create a Deployment:
  ```bash
  kubectl create deployment clock --image=ahmedgabercod/clock
  ```

- Look at its annotations and labels:
  ```bash
  kubectl describe deployment clock
  ```

]

So, what do we get?

---

## Labels and annotations for our Deployment

- We see one label:
  ```
  Labels: app=clock
  ```

- This is added by `kubectl create deployment`

- And one annotation:
  ```
  Annotations: deployment.kubernetes.io/revision: 1
  ```

- This is to keep track of successive versions when doing rolling updates

---

## And for the related Pod?

- Let's look up the Pod that was created and check it too

.exercise[

- Find the name of the Pod:
  ```bash
  kubectl get pods
  ```

- Display its information:
  ```bash
  kubectl describe pod clock-xxxxxxxxxx-yyyyy
  ```

]

So, what do we get?

---

## Labels and annotations for our Pod

- We see two labels:
  ```
    Labels: app=clock
            pod-template-hash=xxxxxxxxxx
  ```

- `app=clock` comes from `kubectl create deployment` too

- `pod-template-hash` was assigned by the Replica Set

  (when we will do rolling updates, each set of Pods will have a different hash)

- There are no annotations:
  ```
  Annotations: <none>
  ```

---

## Selectors

- A *selector* is an expression matching labels

- It will restrict a command to the objects matching *at least* all these labels

.exercise[

- List all the pods with at least `app=clock`:
  ```bash
  kubectl get pods --selector=app=clock
  ```

- List all the pods with a label `app`, regardless of its value:
  ```bash
  kubectl get pods --selector=app
  ```

]

---

## Settings labels and annotations

- The easiest method is to use `kubectl label` and `kubectl annotate`

.exercise[

- Set a label on the `clock` Deployment:
  ```bash
  kubectl label deployment clock color=blue
  ```

- Check it out:
  ```bash
  kubectl describe deployment clock
  ```

]

---

## Other ways to view labels

- `kubectl get` gives us a couple of useful flags to check labels

- `kubectl get --show-labels` shows all labels

- `kubectl get -L xyz` shows the value of label `xyz`

.exercise[

- List all the labels that we have on pods:
  ```bash
  kubectl get pods --show-labels
  ```

- List the value of label `app` on these pods:
  ```bash
  kubectl get pods -L app
  ```

]

---

class: extra-details

## More on selectors

- If a selector has multiple labels, it means "match at least these labels"

  Example: `--selector=app=frontend,release=prod`

- `--selector` can be abbreviated as `-l` (for **l**abels)

  We can also use negative selectors

  Example: `--selector=app!=clock`

- Selectors can be used with most `kubectl` commands

  Examples: `kubectl delete`, `kubectl label`, ...
