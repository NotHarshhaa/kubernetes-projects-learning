# How to deploy DaemonSets Service in Kubernetes (K8s)?

![](https://miro.medium.com/v2/resize:fit:736/1*2iUgGBUrG4EHz3VbJgAQWA.jpeg)

**DaemonSets in Kubernetes Cluster**

Like other controllers, DaemonSets manage groups of replicated Pods.

However, DaemonSet ensures that all or selected Worker Nodes run a copy of a Pod (one-Pod-per-node).

As you add nodes, DaemonSets automatically add Pods to the new nodes. As the nodes are removed from the cluster, those Pods are garbage collected.

Here is the manifest of DaemonSet:

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
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
```

**Create a daemonset:**

```bash
kubectl create -f daemonset.yamldaemonset.apps "fluentd" created
```

**Check the pod running:**

```bash
kubectl get pods -n kube-systemNAME         READY  STATUS  RESTARTS AGE
fluentd-7svlj 1/1   Running     0   58s
fluentd-kwm4x 1/1   Running     0   58s
fluentd-q64wf 1/1   Running     0   58s
```

**Check no. of nodes:**

```bash
kubectl get nodes
NAME                 STATUS      ROLES        AGE            VERSION
gke-cluster-1-default-pool-a6a57f2a-77wb Ready < none > 6h v1.11.6-gke.2
gke-cluster-1-default-pool-a6a57f2a-xkgz Ready < none > 6h v1.11.6-gke.2
gke-cluster-1-default-pool-a6a57f2a-z2bx Ready < none > 6h v1.11.6-gke.2
```

**Display your daemon sets:**

```bash
kubectl get daemonsets
NAME     DESIRED CURRENT READY UP-TO-DATE   AVAILABLE AGE
fluentd    3        3      3      3           3        5m
```

**Get details of a daemonset:**

```bash
kubectl describe daemonset fluentd
```

**Edit a daemonset:**

```bash
kubectl edit daemonset fluentd
```

**Delete a daemonset:**

```bash
kubectl delete daemonset fluentd
daemonset.apps “fluentd” deleted
```

**Some uses of a DaemonSet are:**

1. running a cluster storage daemon, such as glusterd, ceph, on each node.
    
2. running a logs collection daemon on every node, such as fluentd or logstash.
    
3. running a node monitoring daemon on every node, such as Prometheus Node Exporter, AppDynamics Agent, Datadog agent, New Relic agent, etc.
    

**Running Pods on Only Some Nodes**

If you specify a .spec.template.spec.mode selector, then the DaemonSet controller will create Pods on nodes which match that node selector.

```yaml
spec:
   nodeSelector:
      environment: prod
```

Now, If you mention a .spec.template.spec.affinity, then DaemonSet controller will be creating the Pods on nodes which will be matching the node affinity.

```yaml
spec:
  tolerations:
    - key: node-role.kubernetes.io/master
      effect: NoSchedule
```

In case you will not specify anything, then the controller will be creating Pods on every node of the cluster.

**For Reference**

## [**DaemonSet**](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/?source=post_page-----37d642dcd66f--------------------------------)

### [A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to…](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/?source=post_page-----37d642dcd66f--------------------------------)

[kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/?source=post_page-----37d642dcd66f--------------------------------)

## [**What is Kubernetes DaemonSet? K8s DaemonSets Explained**](https://www.bmc.com/blogs/kubernetes-daemonset/?source=post_page-----37d642dcd66f--------------------------------)

### [In this blog post, we will discuss Kubernetes DaemonSet, including what it’s used for, how to create one, and how to…](https://www.bmc.com/blogs/kubernetes-daemonset/?source=post_page-----37d642dcd66f--------------------------------)

[www.bmc.com](https://www.bmc.com/blogs/kubernetes-daemonset/?source=post_page-----37d642dcd66f--------------------------------)

![](https://miro.medium.com/v2/resize:fit:736/0*cRMUa8dBX1dHpj5L)

### If you like this article, please share with others. ❤️
