# Manage Kubernetes Objects with Kustomize

- Note: **Kustomize** is explicitly part of the current CKAD syllabus (Application Deployment domain). It is built into kubectl (`kubectl apply -k`) — no extra tool to install. Unlike Helm, Kustomize doesn't template: it takes plain YAML files and applies *patches and transformations* on top.

## Kustomize basics

### Given a deployment.yaml and service.yaml, create a kustomization.yaml that applies both in one command — without writing any template syntax

<details><summary>show</summary>
<p>

```bash
vi kustomization.yaml
```

```YAML
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

```bash
# apply everything in the directory at once
kubectl apply -k .
# preview what would be applied
kubectl apply -k . --dry-run=client -o yaml
# or just render: kubectl kustomize .
```

</p>
</details>

### Add a common label `app.kubernetes.io/part-of: myapp` to every resource managed by the kustomization

<details><summary>show</summary>
<p>

```YAML
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
labels:
  - pairs:
      app.kubernetes.io/part-of: myapp
    includeSelectors: false   # true would also inject the label into selectors
```

```bash
kubectl kustomize . # verify the labels were added to Deployment AND Service
```

</p>
</details>

### Generate a ConfigMap from a .env file and a literal value directly in the kustomization

<details><summary>show</summary>
<p>

```YAML
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources: []
configMapGenerator:
  - name: app-config
    literals:
      - APP_MODE=prod
    files:
      - config.properties    # a file in the same directory
```

```bash
kubectl kustomize .
# note: kustomize appends a hash to the ConfigMap name (app-config-XXXXX) and
# automatically updates any Deployment that references it — that's how config
# changes trigger rollouts in a GitOps flow
```

</p>
</details>

### Overlays: create a `base` with the raw manifests, and a `prod` overlay that sets replicas=5 and changes the image tag — without editing the base files

<details><summary>show</summary>
<p>

Directory layout:

```
.
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    └── prod/
        └── kustomization.yaml
```

`overlays/prod/kustomization.yaml`:

```YAML
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
patches:
  - target:
      kind: Deployment
      name: myapp
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: nginx:1.25
```

```bash
kubectl apply -k overlays/prod
kubectl kustomize overlays/prod | grep -E "replicas|image:"  # verify the patch
```

*Exam tip*: the patch `target` uses the resource's kind/name (plus namespace/group if needed). JSON6902 patches (as above) need exact paths; for small edits a strategic-merge patch block is often easier to write:

```YAML
patches:
  - target:
      kind: Deployment
      name: myapp
    patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: myapp
      spec:
        replicas: 5
```

</p>
</details>

### Set the namespace and a name prefix for all resources in a kustomization

<details><summary>show</summary>
<p>

```YAML
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
namespace: production     # sets/overrides namespace on all namespaced resources
namePrefix: prod-         # prod-deployment, prod-service, ...
```

```bash
kubectl kustomize . | grep -E "name:|namespace:"
```

</p>
</details>
