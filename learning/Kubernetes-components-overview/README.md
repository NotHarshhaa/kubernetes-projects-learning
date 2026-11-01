# Kubernetes – Architecture and main components overview

> ℹ️ Terminology note: what used to be called "master nodes" is now officially the **control plane**. Both names appear in older docs and articles.

### Architecture – an overview

In general, Kubernetes cluster components looks like next:

![](https://rtfm.co.ua/wp-content/uploads/2019/07/maxresdefault.jpg)

Or a bit more simple one:

![](https://rtfm.co.ua/wp-content/uploads/2019/07/API-server-overview.png)

The cluster itself consists of one or more *Control Plane (Master) Nodes* and one or more *Worker Nodes*.

#### **Control Plane (Master Node)**

Services running on a Control Plane node are called the “[*Kubernetes Control Plane*](https://kubernetes.io/docs/concepts/overview/components/)”, and the Control Plane is used for administrative tasks only, while containers with your services will be created on a Worker Node(s).

##### Kubernetes core services aka Kubernetes Control Plane

* [Kubernetes Master Components: Etcd, API Server, Controller Manager, and Scheduler](https://medium.com/jorgeacetozi/kubernetes-master-components-etcd-api-server-controller-manager-and-scheduler-3a0179fc8186)


On a Control Plane node, there are three main Kubernetes components which make the whole cluster working:

* [`kube-apiserver`](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

    * the main entrypoint for all requests to the cluster, for example `kubectl` commands will be sent as API requests to the [`kube-apiserver`](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

    * the API server serves all REST requests, validates them, authenticates and authorizes them, and persists the objects into `etcd` (the API server is the **only** component which talks to `etcd` – all other components speak to the API server itself, see [*Components interaction*](#components-interaction))

* [`kube-scheduler`](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)

    * makes a decision about which worker node will be used for a new Pod creation (see [*Pod*](#pod)) depending on resources requested, taints/tolerations, affinity rules and node usage

* [`kube-controller-manager`](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)

    * a daemon for [Controllers](https://kubernetes.io/docs/concepts/overview/components/#kube-controller-manager) such as the *Node Controller*, *Job Controller*, *Endpoints Controller*, etc.

    * periodically compares the cluster state via the API server and applies necessary changes — this is the **reconciliation loop**

    * also runs node-lifecycle management and garbage collection


##### `etcd`

* [etcd.io](https://etcd.io/)


Is a *key:value* storage used by Kubernetes as the single source of truth for all cluster data.

Also, it keeps the cluster’s *current* and *desired* states: if K8s finds discrepancies between those states – it will apply changes to make the *current* state match the *desired* state.

> 💡 `etcd` data is the cluster — back it up. `etcdctl snapshot save` is a classic CKA exam task for a reason.

#### **Worker Node**

Worker Node (previously known as a *minion*) – a virtual or bare-metal server with Kubernetes components to create and manage Pods (see [*Pod*](#pod)).

Those components are:

* [`kubelet`](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/): the main Kubernetes component on each cluster node which watches the API server for Pods assigned to its node

    * it communicates with the container runtime ([`containerd`](https://containerd.io/) or [`CRI-O`](https://cri-o.io/)) via the [CRI](https://kubernetes.io/docs/concepts/architecture/cri/) to create and manage containers — the once-popular `rkt` is long dead and Docker as a direct runtime (`dockershim`) was removed in 1.24

    * after any changes in a Pod on a Node – will report them to the API server, which in its turn will save them to the `etcd` database

    * performs container and node health monitoring (and reports `NodeNotReady` conditions)

* [`kube-proxy`](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/): a network proxy running on every node that forwards requests to the appropriate Service or applications inside the Kubernetes private network

    * uses [iptables](https://kubernetes.io/docs/concepts/services-networking/service/#proxy-modes) or IPVS to implement the Service routing rules (you can check existing rules with `kubectl -n kube-system exec -ti <kube-proxy-pod> -- iptables-save -t nat`)

    * see [Understanding Kubernetes Kube-Proxy](https://supergiant.io/blog/understanding-kubernetes-kube-proxy/)


#### **Components interaction**

Example when a new Pod is created:

1. `kubectl` will send a request to the API server

2. the API server will validate it, authenticate/authorize it, and write it to `etcd`

3. `etcd` will reply to the API server that the request was accepted and saved

4. the API server (via the watch mechanism) notifies the `kube-scheduler`

5. `kube-scheduler` will choose a Worker Node for the new Pod and sends this binding back to the API server

6. the API server will send this information to `etcd`

7. `etcd` will reply it accepted and saved the data

8. the `kubelet` on the chosen Worker Node — which watches the API server — sees the Pod assigned to its node

9. `kubelet` will talk to the container runtime ([containerd](https://containerd.io/), [CRI-O](https://cri-o.io/), ...) via the CRI to create a new container

10. `kubelet` will send information about the new Pod's status back to the API server

11. the API server will update the information in `etcd`


### **Kubernetes abstractions**

Above we spoke about more or less “touchable” things such as virtual machines, networks, IP-addresses and so on.

But the Kubernetes itself is just a big piece of an… abstraction, placed upon a physical or virtual infrastructure.

Thus, Kubernetes has a lot of own objects which are abstract or logical Kubernetes components.

#### **Pod**

The Pod – main logical unit in a Kubernetes cluster.

Inherently, a Pod is kind of a virtual machine inside the Kubernetes cluster: it has its own private IP, hostname, shared volumes etc (see [*Volumes*](#volumes)).

A Pod is a deployment unit and inside of this “virtual machine” one or more containers will be created, which are tied by a common goal and which are a logical application with one or more processes running.

Each such Pod is designated to run and serve only one copy of an application: if you want horizontal scaling – you need multiple Pods, managed by a controller like a `Deployment` or `ReplicaSet`.

![](https://rtfm.co.ua/wp-content/uploads/2019/07/Screenshot_20190719_141733.png)

Such a group of Pods is called *Replicated Pods* and is managed by a dedicated controller (see [*Controllers*](#controllers)).

Containers themselves are **not** Kubernetes cluster objects and they are not managed by Kubernetes directly — instead, Kubernetes manages Pods, while containers inside of a Pod share its namespaces including IP address and ports and can communicate with each other via *localhost*.

A Pod’s template example (this file exists in this folder as `my-pod.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
    type: front-app
spec:
  containers:
  - name: nginx-container
    image: nginx
```

### **Services**

**Links**:

* [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

* [Kubernetes Services: A Beginner’s Guide](https://www.bmc.com/blogs/kubernetes-services/)

* [Kubernetes – Services Explained](https://www.youtube.com/watch?v=5lzUpDtmWgM)


Services in the first turn are everything about networking in a Kubernetes cluster.

They are used for communications between an application’s components inside and outside of it.

Basically, services are the same Kubernetes objects as [*Pod*](#pod), [*ReplicaSet*](#replicaset), [*DaemonSet*](#daemonset) are, and you can imagine a Service like a dedicated virtual load-balancer inside of the cluster.

They can be displayed as the next:

![](https://rtfm.co.ua/wp-content/uploads/2019/07/Screenshot_20190719_161337.png)

Here is a user who connects to a frontend application via one Service, then this frontend talks to two backend applications using two additional Services, and backends communicate to a database service via another one Service.

##### `ClusterIP`

Will open access to an application via a cluster’s internal IP, thus it will be accessible only from within the cluster itself.

Is the default Service type.

##### `NodePort`

This Service type will open access to an application using a static port (in the **30000–32767** range) on every node’s IP.

Also, it will automatically create a `ClusterIP` service for the application to route traffic from the `NodePort`.

![](https://rtfm.co.ua/wp-content/uploads/2019/07/Screenshot_20190722_110641.png)

Here:

* *30008* – an external port on a Worker Node which can be used to connect to the `NodePort` service, must be in the 30000–32767 range

* `NodePort` service with the `ClusterIP`, own port (*Port*) and IP from the `serviceSubnet` block

* Pod with an application inside – Pod will accept new connections to its port 80 (*TargetPort*) and has an IP from the `podSubnet` block


Those networks can be checked with `kubectl cluster-info dump | grep -m1 cluster-cidr`, or on a kubeadm cluster (modern kubeadm removed `kubeadm config view`; use):

```bash
kubeadm config print init-defaults | grep -A3 networking
```

And again – you can imagine a Service like another one VM inside of your Worker Node, in the same way as about Pods.

A `NodePort` service’s template example (this file exists in this folder as `my-svc.yaml`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: my-app
    type: front-app
```

##### Service, Pod, labels, selectors

* [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)


To make a Service distinguish which Pods have to be used to route traffic to – the `Labels` and `Selectors` are used.

In the pod’s template example above we added labels for our application:

```yaml
...
  labels:
    app: my-app
    type: front-app
```

Then in a Service’s description – we are using *selectors*:

```yaml
...
  selector:
    app: my-app
```

Thus if a cluster has multiple Pods with such a label – then a Service will try to route traffic to all of them:

![](https://rtfm.co.ua/wp-content/uploads/2019/07/Screenshot_20190722_113024.png)

In case if an application is placed on multiple Worker Nodes – then a `NodePort` service will be spread between all those nodes and the *30008* port will be opened on every such a node.

Thus, you can access an application via a `NodePort` service using the IP of **any** Worker Node:

![](https://rtfm.co.ua/wp-content/uploads/2019/07/Screenshot_20190722_113433.png)

##### `LoadBalancer`

* [Type LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)

* [Cloud Providers](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/)

* [Using Kubernetes LoadBalancer Services on AWS](https://blog.giantswarm.io/load-balancer-service-use-cases-on-aws/)


Will create a cloud provider’s Load Balancer, for example – [AWS NLB/ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html).

Worker Nodes will be attached to this Load Balancer, and traffic will be routed via the internal `LoadBalancer` service to a node’s `NodePort` service.

`NodePort` and `ClusterIP` will be created automatically.

On bare metal (no cloud LB) consider [MetalLB](https://metallb.universe.tf/).

##### `ExternalName`

Will map the Service to the contents of its `externalName` field by returning a CNAME DNS record — it does **not** proxy traffic and has no selectors.

An `ExternalName` service template example (this file exists in this folder as `service.yaml`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-google-svc
spec:
  type: ExternalName
  externalName: google.com
```

After it will be created – you can reach it from any Pod via this Service name, for example, *my-google-svc* as per the template example above:

```bash
kubectl exec -it my-pod -- nslookup my-google-svc.default.svc.cluster.local
```

#### Volumes

* [Volumes](https://kubernetes.io/docs/concepts/storage/volumes)

* [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

* [Kubernetes Volumes Guide](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-volumes-example-nfs-persistent-volume.html)


Data in containers is ephemeral, i.e. if a container will be recreated by the `kubelet` – then all data from the old container will be lost.

Also, multiple containers inside of the same Pod can use shared data.

The `Volumes` concept in Kubernetes is similar to the Docker’s solution but with much more features.

For example, Kubernetes Volumes support various drivers to mount volumes to Pods, like [`hostPath`](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath), [`nfs`](https://kubernetes.io/docs/concepts/storage/volumes/#nfs), [`emptyDir`](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir), etc. (the old in-tree cloud driver types like `awsElasticBlockStore` are deprecated — on cloud clusters use the [CSI](https://kubernetes.io/docs/concepts/storage/volumes/#csi) volumes / StorageClasses instead).

See [Storage & Persistent Volumes](../Storage-PersistentVolumes/README.md) for a hands-on guide.

#### Namespaces

* [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

* [Share a Cluster with Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)


*Namespace* in Kubernetes is kind of a dedicated cluster inside of your existing cluster with its own set of scopes for networking, disks/volumes, processes and so on.

The main idea behind Namespaces is to dedicate working environments for teams and workloads, and they can be used to set resource usage limits – CPU, memory, etc. See [*Resource Quotas*](https://kubernetes.io/docs/concepts/policy/resource-quotas/).

Namespaces are also used by the Kubernetes DNS service to create a URL in the *<name>.<namespace>.svc.cluster.local* form.

Most of the Kubernetes resources live in such Namespaces, you can list them with:

```bash
kubectl api-resources --namespaced=true
```

To list resources living outside of any Namespace – use the next command:

```bash
kubectl api-resources --namespaced=false
```

#### Controllers

A *Controller* in Kubernetes is a continuously working process, which communicates with the API server, checks the current state of a cluster and makes necessary changes to make the *current* state equal to the *desired* state.

Besides the standard Controllers – you can create your own, see [How to Create a Kubernetes Custom Controller Using client-go](https://itnext.io/how-to-create-a-kubernetes-custom-controller-using-client-go-f36a7a7536cc).

##### `ReplicaSet`

* [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

* [Key Kubernetes Concepts](https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e)

* [Kubernetes KnowHow – Working With ReplicaSet](https://dzone.com/articles/kubernetes-knowhow-working-with-replicaset)


`ReplicaSet` is created by the `Deployment` and the main goal of the `ReplicaSet` is to create and scale Pods.

`ReplicaSet` is the next generation of the [`ReplicationController`](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) and can use set-based selectors (see [Service, Pod, labels, selectors](#service-pod-labels-selectors)).

It’s recommended to use `Deployment` instead of creating `ReplicaSet` objects directly.

A `ReplicaSet` template example (this file exists in this folder as `my-nginx-rc.yaml`):

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-nginx-rc
  labels:
    app: my-nginx-rc-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx-rc-app
  template: # PODs template
    metadata:
      labels:
        app: my-nginx-rc-app
    spec:
      containers:
      - name: nginx-container
        image: nginx
```

##### `Deployment`

* [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

* [Kubernetes Deployments: The Ultimate Guide](https://semaphoreci.com/blog/kubernetes-deployment/)

* [Kubernetes Deployment Tutorial For Beginners](https://devopscube.com/kubernetes-deployment-tutorial/)

* [K8s: Deployments vs StatefulSets vs DaemonSets](https://medium.com/stakater/k8s-deployments-vs-statefulsets-vs-daemonsets-60582f0c62d4)


The *Deployment* controller will apply changes to Pods and `ReplicaSets` and currently is the most used resource in Kubernetes to deploy applications.

It’s mainly used for *stateless* applications, but you can attach a [*Persistent Volume*](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) and use it for a *stateful* application — although `StatefulSet` is the proper controller for that.

See [stateful](https://whatis.techtarget.com/definition/stateful-app) vs [stateless](https://whatis.techtarget.com/definition/stateless-app).

During a `Deployment` creation – it will create a `ReplicaSet` object which in its turn will create and manage Pods for this `Deployment`.

`Deployment` used to:

* update Pods – `Deployment` will create a new `ReplicaSet` and will update a deployment’s revision number (`deployment.kubernetes.io/revision`)

* roll back a deployment if it was unsuccessful using revisions (`kubectl rollout undo`)

* scale Pods (`kubectl scale` and `kubectl autoscale`, see [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/))

* can be used for canary deployments (see the [Intro to deployment strategies: blue-green, canary, and more](https://dev.to/mostlyjason/intro-to-deployment-strategies-blue-green-canary-and-more-3a3))


[![](https://res.cloudinary.com/practicaldev/image/fetch/s--to_dDoP6--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://rtfm.co.ua/wp-content/uploads/2019/07/07751442-deployment.png)](https://rtfm.co.ua/wp-content/uploads/2019/07/07751442-deployment.png)

The old `kubectl rolling-update` command is deprecated — use `Deployment` updates instead.

For autoscaling based on CPU/memory metrics see [Autoscaling (HPA)](../Autoscaling-HPA/README.md).

##### `StatefulSet`

* [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)


`StatefulSet` is used to manage stateful applications.

It will create Pods with unique, stable names (`web-0`, `web-1`, ...) directly instead of going through a `ReplicaSet`. Because of this, when using a `StatefulSet` you have no `Deployment`-style rollout undo. Instead, you can delete a resource or scale it.

During `StatefulSet` updates – an [`OnDelete` or `RollingUpdate`](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#update-strategies) strategy will be applied to all Pods.

See a full working example in [Kubernetes for Everyone](../kubernetes-for-everyone/README.md).

##### `DaemonSet`

* [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)


`DaemonSet` is used when you need to run an application on **all** (or selected) nodes in your cluster – not only Worker Nodes. When a **new Node** is added to the cluster after the `DaemonSet` – a copy of the Pod will be deployed on this new node automatically; when a node is removed, its Pod is garbage collected.

`DaemonSet` is the perfect decision to run applications which have to be present on every node, for example – monitoring agents, log collectors, CNI plugins, kube-proxy itself, etc.

During this, some nodes will refuse to create Pods on them, such as a Control Plane node, because it has a `NoSchedule` taint set (see [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)):

```bash
kubectl describe node <control-plane-node> | grep Taint
# Taints:  node-role.kubernetes.io/control-plane:NoSchedule   (older clusters: node-role.kubernetes.io/master:NoSchedule)
```

Respectively, when creating a `DaemonSet` which has to create Pods on the Control Plane nodes too you have to specify `tolerations`.

Such a `DaemonSet` template example (this file exists in this folder as `my-nginx-ds.yaml`):

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-nginx-ds
spec:
  selector:
    matchLabels:
      app: my-nginx-pod
  template:
    metadata:
      labels:
        app: my-nginx-pod
    spec:
      tolerations:
      - effect: NoSchedule
        operator: Exists
      containers:
      - name: nginx-container
        image: nginx
        ports:
        - containerPort: 80
```

During `DaemonSet` update – the [`RollingUpdate` strategy](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#daemonset-update-strategy) for all pods will be applied.

A hands-on guide with commands lives in [Deploy DaemonSets](../Deploy-DaemonSets-Service-in-Kubernetes/README.md).

##### `Job` and `CronJob`

* [Jobs – Run to Completion](https://kubernetes.io/docs/concepts/workloads/controllers/job/run-to-completion/)

* [Working with Kubernetes Jobs](https://medium.com/coryodaniel/working-with-kubernetes-jobs-848914418)


A `Job` in Kubernetes is intended to create a Pod which will execute one task, and once it finishes – the Pod is stopped.

A `Job` can create one or more Pods: `completions` sets how many successful Pods you need in total, `parallelism` how many may run at once, and `backoffLimit` how many failed attempts are tolerated before the Job itself is marked failed.

A `Job` template example (this file exists in this folder as `job-example.yaml`):

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-example
spec:
  completions: 2
  parallelism: 2
  backoffLimit: 4
  template:
    metadata:
      name: counter
    spec:
      containers:
      - name: counter
        image: ubuntu:22.04
        command: ["bash"]
        args: ["-c",  "for i in {1..10}; do echo $i; done"]
      restartPolicy: Never    # Jobs require OnFailure or Never
```

###### `CronJob`

Similar to the `Job` but intended to run tasks by a schedule – check the `schedule` parameter below:

> ⚠️ `apiVersion: batch/v1beta1` was removed in Kubernetes 1.25 — always use `batch/v1`.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-example
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      completions: 2
      parallelism: 2
      template:
        metadata:
          name: counter
        spec:
          containers:
          - name: counter
            image: ubuntu:22.04
            command: ["bash"]
            args: ["-c",  "for i in {1..10}; do echo $i; done"]
          restartPolicy: Never
```

### **Useful links**

#### **Common**

* [Concepts](https://kubernetes.io/docs/concepts/)

* [Key Kubernetes Concepts](https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e)

* [Kubernetes Master Components: Etcd, API Server, Controller Manager, and Scheduler](https://medium.com/jorgeacetozi/kubernetes-master-components-etcd-api-server-controller-manager-and-scheduler-3a0179fc8186)

* [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)

* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

* [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)


#### **Networking**

* [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

* [Cloud Providers](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/)

* [Understanding Kubernetes Kube-Proxy](https://supergiant.io/blog/understanding-kubernetes-kube-proxy/)

* [Kubernetes Services: A Beginner’s Guide](https://www.bmc.com/blogs/kubernetes-services/)

* [Kubernetes – Services Explained](https://www.youtube.com/watch?v=5lzUpDtmWgM)

* [Using Kubernetes LoadBalancer Services on AWS](https://blog.giantswarm.io/load-balancer-service-use-cases-on-aws/)


#### **Deployments**

* [Kubernetes Deployment Strategies](https://www.weave.works/blog/kubernetes-deployment-strategies)

* [Kubernetes Deployments: The Ultimate Guide](https://semaphoreci.com/blog/kubernetes-deployment/)

* [Kubernetes Deployment Tutorial For Beginners](https://devopscube.com/kubernetes-deployment-tutorial/)

* [K8s: Deployments vs StatefulSets vs DaemonSets](https://medium.com/stakater/k8s-deployments-vs-statefulsets-vs-daemonsets-60582f0c62d4)

### If you like this article, please share with others. ❤️
