# Helm 3

- For larger stacks, managing thousands of lines of YAML is unreasonable

- These YAML bundles need to be customized with variable parameters

  (E.g.: number of replicas, image version to use ...)

- It would be nice to be able to upgrade/rollback these bundles carefully

- [Helm](https://helm.sh/) 

---

## Helm concepts

- `helm` is a CLI tool

- It is used to find, install, upgrade *charts*

- A chart is an archive containing templatized YAML bundles

- Charts are versioned

- Charts can be stored on private or public repositories

---

## Installing Helm

- If the `helm` CLI is not installed in your environment, install it

.exercise[

- Check if `helm` is installed:
  ```bash
  helm
  ```

- If it's not installed, run the following command:
  ```bash
  curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get-helm-3 \
  | bash
  ```

  ```bash
    choco install kubernetes-helm
  ```

]


---

## Managing repositories

- Let's check what repositories we have, and add the `stable` repo

  (the `stable` repo contains a set of official-ish charts)

.exercise[

- List our repos:
  ```bash
  helm repo list
  ```

- Add the `stable` repo:
  ```bash
  helm repo add bitnami https://charts.bitnami.com/bitnami 
  ```

]

Adding a repo can take a few seconds (it downloads the list of charts from the repo).

It's OK to add a repo that already exists (it will merely update it).

---

## Search available charts

- We can search available charts with `helm search`

- We need to specify where to search (only our repos, or Helm Hub)

- Let's search for all charts mentioning tomcat!

.exercise[

- Search for nginx in the repo that we added earlier:
  ```bash
  helm search repo nginx
  ```
]

---

## Charts and releases

- "Installing a chart" means creating a *release*

- We need to name that release

  (or use the `--generate-name` to get Helm to generate one for us)

.exercise[

- Install the tomcat chart that we found earlier:
  ```bash
  helm install nginx bitnami/nginx
  ```

- List the releases:
  ```bash
  helm list
  ```

]
