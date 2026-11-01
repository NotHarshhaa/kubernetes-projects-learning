# Autoscaling in Kubernetes (HPA)

Manually scaling deployments (`kubectl scale`) doesn't survive real traffic patterns. The **HorizontalPodAutoscaler (HPA)** automatically changes the number of Pod replicas based on observed metrics — the standard answer to "scale out on load, scale in to save money".

> **Horizontal** = more/fewer Pods. **Vertical** (VPA) = more/fewer CPU/memory per Pod. **Cluster** autoscaling = more/fewer Nodes. HPA is the horizontal one.

## How it works

```
metrics-server ──► CPU/memory usage per pod
                       │
                       ▼
        HPA controller (every ~15s):
        desiredReplicas = ceil(currentReplicas × currentMetric / targetMetric)
                       │
                       ▼
             updates Deployment .spec.replicas
```

For the `Utilization` target type, HPA compares actual CPU usage against the **requested** CPU — which is why resource requests are mandatory (see `deployment-for-hpa.yaml`).

## Prerequisites

HPA reads metrics from the **metrics-server** (not Prometheus) — it must be installed:

```bash
minikube addons enable metrics-server

# verify it is serving
kubectl top nodes
kubectl top pods
```

## Files in this folder

| File | Purpose |
|------|---------|
| `deployment-for-hpa.yaml` | Deployment with the resource requests HPA needs |
| `hpa.yaml` | HPA targeting 60% CPU + 200Mi memory, 2–10 replicas |

## Walkthrough

### 1. Deploy

```bash
kubectl apply -f deployment-for-hpa.yaml
kubectl apply -f hpa.yaml

kubectl get hpa
# NAME          REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS
# web-app-hpa   Deployment/web-app   1%/60%    2         10        2
```

> If `TARGETS` shows `<unknown>/60%`, metrics-server isn't working yet — give it a minute and check `kubectl describe hpa web-app-hpa`.

### 2. Generate load

```bash
kubectl run load-generator --rm -it --image=busybox:1.36 --restart=Never \
  -- /bin/sh -c "while true; do wget -q -O- http://web-app; done"
```

### 3. Watch it scale

```bash
kubectl get hpa web-app-hpa -w
kubectl get pods -l app=web-app -w
```

You'll see replicas climb toward 10 while load runs, and drift back down ~5 minutes after it stops (the scale-down stabilization window).

## The autoscaling/v2 spec, field by field

| Field | Meaning |
|-------|---------|
| `scaleTargetRef` | The workload to scale (Deployment, ReplicaSet, StatefulSet) |
| `minReplicas` / `maxReplicas` | Hard bounds — HPA never scales outside them |
| `metrics[].resource.name` | `cpu` or `memory` |
| `target.type: Utilization` | % of the **requested** amount (needs requests set) |
| `target.type: AverageValue` | Absolute per-pod value (e.g. `200Mi`) — required for memory |
| `behavior` | Optional scale-up/down speed limits and stabilization |

Multiple metrics can be combined: HPA computes the replica count for each and takes the **maximum**.

## Common pitfalls

1. **No resource requests on containers** → HPA can't compute utilization and silently does nothing (`kubectl describe hpa` shows `missing request for cpu`).
2. **HPA + `kubectl scale` fight** → direct scaling is ignored; change `minReplicas`/`maxReplicas` instead.
3. **Scaling too fast down** → use `behavior.scaleDown.stabilizationWindowSeconds` to survive traffic spikes that dip briefly.
4. **Expecting instant scaling** → metrics arrive ~every 15s and pods take time to start; HPA is for load over minutes, not millisecond bursts.
5. **Scaling pods but not nodes** → if the cluster has no room, pods stay `Pending` — pair HPA with a cluster autoscaler.

## Beyond CPU/memory

- **Custom & external metrics** (queue depth, requests/sec): needs a custom-metrics adapter — [Prometheus Adapter](https://github.com/kubernetes-sigs/prometheus-adapter) or [KEDA](https://keda.sh/) (which can also scale to zero).
- **VPA** ([vertical-pod-autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)) recommends/adjusts requests&limits. Don't run VPA and HPA on the same CPU/memory metrics of the same workload.
- **Cluster Autoscaler / Karpenter** add and remove nodes so HPA'd pods have somewhere to run.

## Additional Resources

- [HorizontalPodAutoscaler Concepts](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [autoscaling/v2 API Reference](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/horizontal-pod-autoscaler-v2/)
- [metrics-server](https://github.com/kubernetes-sigs/metrics-server)
- [KEDA — event-driven autoscaling](https://keda.sh/)
- [Karpenter — node autoscaling](https://karpenter.sh/)
