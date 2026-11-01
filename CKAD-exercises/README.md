[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

# CKAD Exercises

A set of exercises that help you prepare for the [Certified Kubernetes Application Developer (CKAD)](https://www.cncf.io/training/certification/ckad/) exam, offered by the Cloud Native Computing Foundation, organized by curriculum domain.
They may as well serve as learning material and practice for Kubernetes in general.

## Exam tips 🎯

- **Make a mental note of the breadcrumb** at the start of each exercise section — it locates the relevant document on [kubernetes.io](https://kubernetes.io), which you are allowed to browse **in a single tab of the exam VM** (no other sites, no personal notes).
- **Practice the imperative one-liners.** ~70% of exam tasks are solvable faster with `kubectl run/create/expose/set/scale/autoscale --dry-run=client -o yaml` than by writing YAML from scratch. Use `--dry-run=client -o yaml | kubectl apply -f -` or `> file.yaml` + `vi` when you need to edit.
- **Set up speed aliases the moment the exam starts** (allowed and recommended):

  ```bash
  alias k=kubectl
  export do="--dry-run=client -o yaml"      # for quick YAML generation
  export now="--force --grace-period=0"     # for fast deletes
  export wn="-o jsonpath='{.spec.containers[*].name}'"  # whatever you repeat often
  ```

- **Use `kubectl explain` liberally** (`kubectl explain pod.spec.containers.livenessProbe --recursive`) when you can't remember a field name.
- **Watch your time:** check the task weight (bottom-right of each question) and don't sink minutes into a 2% question. Flag it and come back.
- **Verify before moving on:** `kubectl get <resource> -o yaml`, `kubectl describe`, or `kubectl run` a temp busybox to test connectivity.
- Practice on [killer.sh](https://killer.sh/ckad) (2 free sessions included with exam purchase) — its UI is nearly identical to the real exam.

## Contents — mapped to the current CKAD curriculum (v1.37)

> Weights below are from the [official CNCF curriculum](https://github.com/cncf/curriculum) (CKAD_Curriculum_v1.37). CNCF revises this roughly with each Kubernetes release — always double-check the latest PDF before you book the exam.

| Curriculum domain (weight) | Sub-topics | Exercises |
|---|---|---|
| **Application Design and Build (20%)** | Container images • workload resources (Deployment, DaemonSet, CronJob…) • multi-container patterns (sidecar, init) • persistent & ephemeral volumes | [a.core_concepts.md](a.core_concepts.md) • [b.multi_container_pods.md](b.multi_container_pods.md) • [g.state.md](g.state.md) • [j.podman.md](j.podman.md) |
| **Application Deployment (20%)** | Blue/green & canary • Deployments & rolling updates • Helm • **Kustomize** | [c.pod_design.md](c.pod_design.md) • [h.helm.md](h.helm.md) • [k.kustomize.md](k.kustomize.md) |
| **Application Observability and Maintenance (15%)** | API deprecations • probes & health checks • CLI monitoring • container logs • debugging | [e.observability.md](e.observability.md) |
| **Application Environment, Configuration and Security (25%)** | CRDs & Operators • authn/authz/admission • requests, limits, quotas • ConfigMaps • Secrets • ServiceAccounts • SecurityContexts | [d.configuration.md](d.configuration.md) • [i.crd.md](i.crd.md) |
| **Services and Networking (20%)** | NetworkPolicies • services & troubleshooting access • **Ingress rules** | [f.services.md](f.services.md) |

<details><summary>Legacy file→domain mapping (old 2020 syllabus, kept for reference)</summary>

- Core Concepts 13% → a.core_concepts.md
- Multi-container pods 10% → b.multi_container_pods.md
- Pod design 20% → c.pod_design.md
- Configuration 18% → d.configuration.md
- Observability 18% → e.observability.md
- Services and networking 13% → f.services.md
- State persistence 8% → g.state.md

</details>

## Quick reference box

| Topic | Breadcrumb on kubernetes.io |
|---|---|
| kubectl imperative commands | Docs > Reference > [Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/) |
| Probes | Docs > Tasks > Configure Pods and Containers > [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) |
| Jobs / CronJobs | Docs > Concepts > Workloads > [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) / [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) |
| Ingress | Docs > Concepts > Services, Load Balancing, and Networking > [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) |
| Kustomize | Docs > Tasks > [Manage Kubernetes Objects with Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/) |
| Helm | Docs > Tasks > [Deploy an Application with Helm](https://kubernetes.io/docs/tasks/tutorials/helm/deploys/) |

Feel free to PR and edit/add questions and solutions, but please stick to the existing format.
