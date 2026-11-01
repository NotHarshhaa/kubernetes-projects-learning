# Helm — the Kubernetes Package Manager

By now you have written a lot of YAML. Helm solves what plain YAML can't: **parameterize it, version it, share it, and upgrade/roll back it as one unit.** A Helm chart is a set of Go-templated manifests plus a `values.yaml` of defaults.

## Files in this folder

```
Helm-Package-Manager/
├── README.md
└── demo-chart/              # a minimal hand-written chart
    ├── Chart.yaml           # chart metadata (name, chart version, app version)
    ├── values.yaml          # default configuration
    └── templates/
        ├── deployment.yaml  # {{ .Values... }} placeholders filled at install time
        └── service.yaml
```

## Install Helm

```bash
# macOS / Linux (brew), or see https://helm.sh/docs/intro/install/
brew install helm
# Windows
choco install kubernetes-helm
```

## Walkthrough

### 1. Render without installing (always start here)

```bash
helm template my-release ./demo-chart        # print the final YAML
helm lint ./demo-chart                       # catch mistakes
helm install my-release ./demo-chart --dry-run --debug
```

### 2. Install a release

```bash
helm install my-release ./demo-chart

kubectl get all -l app=demo-app
# NAME                                READY  ...  (2 pods from replicaCount)
# service/my-release-demo-app-svc     ClusterIP
```

A *release* = one named instance of a chart. Install the same chart twice with different names and you get two independent, fully-parameterized deployments.

### 3. Override values

```bash
# one-off settings
helm upgrade my-release ./demo-chart --set replicaCount=4 --set service.type=NodePort

# or a values file (per-environment config — dev.yaml, prod.yaml...)
helm upgrade my-release ./demo-chart -f my-values.yaml
```

### 4. Upgrade & rollback

```bash
# change something in values.yaml, then:
helm upgrade my-release ./demo-chart

helm history my-release          # revision list
helm rollback my-release 1       # instant rollback to revision 1
```

This is Helm's killer feature: every change is a tracked revision with a one-command undo — no more `kubectl apply` archaeology.

### 5. Uninstall

```bash
helm uninstall my-release
```

## Templating cheat sheet

| Template expression | Resolves to |
|---------------------|-------------|
| `{{ .Values.replicaCount }}` | value from `values.yaml` (or `--set`) |
| `{{ .Release.Name }}` | the release name you passed to `helm install` |
| `{{ .Chart.Name }}` / `{{ .Chart.Version }}` | fields from `Chart.yaml` |
| `{{ toYaml .Values.resources \| nindent 10 }}` | inject a whole YAML block, indented |

Values merge in order: `values.yaml` → parent chart values → `-f file.yaml` → `--set` flags (last wins).

## Working with public charts

Most real-world Helm usage is installing *other people's* charts:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack \
  -f prometheus-values.yaml     # see ../Monitoring-and-Logging/

helm search repo nginx          # find charts
helm show values ingress-nginx/ingress-nginx   # inspect what can be configured
```

## Useful commands

```bash
helm list -A                    # all releases, all namespaces
helm get manifest my-release    # what was actually applied
helm get values my-release      # values in effect
helm diff upgrade my-release ./demo-chart   # (helm-diff plugin) preview before upgrading
```

## Helm vs Kustomize

| | Helm | Kustomize |
|---|------|-----------|
| Model | Templates + values, packaged & versioned | Plain YAML + overlays/patches, no templating |
| Best for | Distributing apps, many configurable knobs | Small per-env tweaks of your own manifests |
| Built into kubectl | No (`helm` CLI) | Yes (`kubectl apply -k`) |

They are not enemies — many teams Helm-install base components and Kustomize-patch their own manifests.

## Additional Resources

- [Helm Docs](https://helm.sh/docs/)
- [Chart Template Guide](https://helm.sh/docs/chart_template_guide/getting_started/)
- [Artifact Hub](https://artifacthub.io/) — search thousands of public charts
- [Kustomize](https://kustomize.io/)
