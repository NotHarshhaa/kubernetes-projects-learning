# Managing Kubernetes with Helm

- Note: Helm is part of the new CKAD syllabus. Here are a few examples of using Helm to manage Kubernetes.

## Helm in K8s

### Creating a basic Helm chart

<details><summary>show</summary>
<p>

```bash
helm create chart-test ## this would create a helm 
```

</p>
</details>

### Running a Helm chart

<details><summary>show</summary>
<p>

```bash
helm install -f myvalues.yaml myredis ./redis
```

</p>
</details>

### Find pending Helm deployments on all namespaces

<details><summary>show</summary>
<p>

```bash
helm list --pending -A
```

</p>
</details>

### Uninstall a Helm release

<details><summary>show</summary>
<p>

```bash
helm uninstall -n namespace release_name
```

</p>
</details>

### Upgrading a Helm chart

<details><summary>show</summary>
<p>

```bash
helm upgrade -f myvalues.yaml -f override.yaml redis ./redis
```

</p>
</details>

### Using Helm repo

<details><summary>show</summary>
<p>

Add, list, remove, update and index chart repos

```bash
helm repo add [NAME] [URL]  [flags]

helm repo list / helm repo ls

helm repo remove [REPO1] [flags]

helm repo update / helm repo up

helm repo update [REPO1] [flags]

helm repo index [DIR] [flags]
```

</p>
</details>

### Download a Helm chart from a repository 

<details><summary>show</summary>
<p>

```bash
helm pull [chart URL | repo/chartname] [...] [flags] ## this would download a helm, not install 
helm pull --untar [rep/chartname] # untar the chart after downloading it 
```

</p>
</details>

### Add the Bitnami repo at https://charts.bitnami.com/bitnami to Helm
<details><summary>show</summary>
<p>
    
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```
  
</p>
</details>

### Write the contents of the values.yaml file of the `bitnami/node` chart to standard output
<details><summary>show</summary>
<p>
    
```bash
helm show values bitnami/node
```
  
</p>
</details>

### Install the `bitnami/node` chart setting the number of replicas to 5
<details><summary>show</summary>
<p>

To achieve this, we need two key pieces of information:
- The name of the attribute in values.yaml which controls replica count
- A simple way to set the value of this attribute during installation

To identify the name of the attribute in the values.yaml file, we could get all the values, as in the previous task, and then grep to find attributes matching the pattern `replica`
```bash
helm show values bitnami/node | grep -i replica
```
which returns
```bash
## @param replicaCount Specify the number of replicas for the application
replicaCount: 1
```
 
We can use the `--set` argument during installation to override attribute values. Hence, to set the replica count to 5, we need to run
```bash
helm install mynode bitnami/node --set replicaCount=5
```

</p>
</details>



### List all releases in all namespaces and see the details (values, manifest) of one

<details><summary>show</summary>
<p>

```bash
helm list -A                 # all releases, all namespaces
helm get values <release> -n <namespace>       # values in effect
helm get manifest <release> -n <namespace>     # what was actually applied to the cluster
helm status <release> -n <namespace>
```

</p>
</details>

### Roll back a release to its previous revision (and to a specific one)

<details><summary>show</summary>
<p>

```bash
helm history <release> -n <namespace>          # list revisions
helm rollback <release> -n <namespace>         # back to the previous revision
helm rollback <release> 1 -n <namespace>       # back to a specific revision number
```

*Note*: `helm rollback` re-applies the old release manifest — it does not revert data written by the app into volumes.

</p>
</details>

### Render a chart locally without installing it, and check it for issues

<details><summary>show</summary>
<p>

```bash
helm template myrelease ./mychart             # print the final rendered YAML
helm lint ./mychart                           # catch mistakes
helm install myrelease ./mychart --dry-run --debug   # simulate an install
```

</p>
</details>

### Search for a chart on Artifact Hub and show its default values before installing

<details><summary>show</summary>
<p>

```bash
helm search hub nginx                         # search Artifact Hub
helm search repo bitnami/nginx --versions     # search added repos, with versions
helm show values bitnami/nginx                # inspect the defaults
helm show chart bitnami/nginx                 # inspect Chart.yaml metadata
```

</p>
</details>
