# How to deploy DaemonSets in Kubernetes (K8s)?

![](https://miro.medium.com/v2/resize:fit:736/1*2iUgGBUrG4EHz3VbJgAQWA.jpeg)

**DaemonSets in a Kubernetes Cluster**

Like other controllers, DaemonSets manage groups of replicated Pods.

However, a DaemonSet ensures that all (or selected) nodes run exactly one copy of a Pod (one-Pod-per-node).

As you add nodes, DaemonSets automatically add Pods to the new nodes. As nodes are removed from the cluster, those Pods are garbage collected.

> 💡 DaemonSets ignore the `replicas` field entirely — the replica count always equals the number of matching nodes.

Here is the manifest of a DaemonSet (this file exists in this folder as `fluentd.yaml`):

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd          # must match the template labels below
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.16-debian-1
```

**Create a DaemonSet:**

```bash
kubectl create -f daemonset.yaml
# daemonset.apps "fluentd" created
```

**Check the pods running:**

```bash
kubectl get pods -n kube-system
# NAME          READY  STATUS   RESTARTS  AGE
# fluentd-7svlj 1/1    Running  0         58s
# fluentd-kwm4x 1/1    Running  0         58s
# fluentd-q64wf 1/1    Running  0         58s
```

**Check the number of nodes** (one fluentd pod per node):

```bash
kubectl get nodes
# NAME                                    STATUS   ROLES    AGE   VERSION
# gke-cluster-1-default-pool-a6a57f2a-77wb Ready   <none>   6h    v1.11.6-gke.2
# gke-cluster-1-default-pool-a6a57f2a-xkgz Ready   <none>   6h    v1.11.6-gke.2
# gke-cluster-1-default-pool-a6a57f2a-z2bx Ready   <none>   6h    v1.11.6-gke.2
```

**Display your DaemonSets:**

```bash
kubectl get daemonsets
# NAME      DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  AGE
# fluentd   3        3        3      3           3          5m
```

Note how `DESIRED` always matches the node count — that's the DaemonSet contract.

**Get details of a DaemonSet:**

```bash
kubectl describe daemonset fluentd
```

**Edit a DaemonSet:**

```bash
kubectl edit daemonset fluentd
```

**Delete a DaemonSet:**

```bash
kubectl delete daemonset fluentd
# daemonset.apps "fluentd" deleted
```

**Some uses of a DaemonSet are:**

1. Running a cluster storage daemon, such as glusterd or ceph, on each node.

2. Running a logs collection daemon on every node, such as fluentd, fluent-bit or logstash.

3. Running a node monitoring daemon on every node, such as Prometheus Node Exporter, Datadog agent, New Relic agent, etc.

4. Running the cluster's own networking: `kube-proxy` and most CNI plugins are themselves DaemonSets.

**Running Pods on only some nodes**

By default the DaemonSet controller creates Pods on every node. To restrict it, use a `nodeSelector` under `.spec.template.spec` — Pods will then only be created on nodes carrying that exact label:

```yaml
spec:
  template:
    spec:
      nodeSelector:
        environment: prod
```

Label the nodes accordingly:

```bash
kubectl label nodes <node-name> environment=prod
```

For richer matching rules (In/NotIn/Exists operators), use `nodeAffinity` instead of `nodeSelector`:

```yaml
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values: [prod, staging]
```

**Running Pods on control-plane nodes too**

Control-plane nodes carry a `NoSchedule` taint, so DaemonSets skip them unless you add matching `tolerations` (see `fluentd.yaml` in this folder):

```yaml
spec:
  template:
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
```

**Additional Resources**

For more information about DaemonSets, check out these resources:

- [Kubernetes DaemonSet Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) — Official Kubernetes documentation explaining DaemonSets in detail
- [Understanding Kubernetes DaemonSets](https://www.bmc.com/blogs/kubernetes-daemonset/) — A comprehensive guide covering DaemonSet use cases and implementation

![DaemonSet Architecture Diagram](https://miro.medium.com/v2/resize:fit:736/0*cRMUa8dBX1dHpj5L)

### If you like this article, please share with others. ❤️
