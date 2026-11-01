# What is a Pod in Kubernetes? ☸️💡🎉

![pod](https://imgur.com/8jx68Ep.png)

# Pods

> A Pod is the basic execution unit of a Kubernetes application–the smallest and simplest unit in the Kubernetes object model that you create or deploy.

We will define a "Hello World" pod. We will define it with a `yaml` file. We will define the `apiVersion` and also instruct the `kind` of Kubernetes object (like Pod, Deployment, Service, etc.,).

```yaml
apiVersion: v1
kind: Pod
```

Then we will have to specify some identifiers for the Pod that we are creating. These are called `metadata`. The `metadata` object holds the name for the pod and any labels to identify the pod.

```yaml
metadata:
  name: hello-world
  labels:
    app: hello
```

We need to define the containers that will run in the Pod. A pod can contain one or more containers inside. We can define them in `spec` object.

```yaml
spec:
  containers:
  - name: hello-world-container
    image: busybox:1.36
    command: ['sh', '-c', 'echo Hello World! && sleep 3600']
```

This is more similar to the Docker definition except the keywords differ. But all it does is get the `busybox` image from the repository and run the specified command. It also names the container to `hello-world-container` when running the image.

[![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--hW4gRlU0--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://thepracticaldev.s3.amazonaws.com/i/bpgn65e0fp83v1lxqhhy.jpeg)](https://res.cloudinary.com/practicaldev/image/fetch/s--hW4gRlU0--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://thepracticaldev.s3.amazonaws.com/i/bpgn65e0fp83v1lxqhhy.jpeg)

## But why a pod and not a container?

Sometimes we need to share the resource between two different containers. They need to connect with each other without any latencies. Making them coexist in a cohesive environment is desirable. We will add these containers in a pod. Containers in the same pod share the same network namespace — the same IP address and port space — which makes it easy for them to share and communicate easily.

These shared containers in a pod will have the following advantages:

1. They communicate with each other using `localhost`.

2. They share the storage volumes

3. They form a cohesive unit of service.


Check out more about Pods [here](https://kubernetes.io/docs/concepts/workloads/pods/).

---

## Init and App container

Kubernetes schedules the pods and runs them inside the cluster. Sometimes we will need to run something before a container is started.

For example, we need to start the application only after the Database is started. We will use `initContainers`.

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--HK4oGCmu--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_800/https://thepracticaldev.s3.amazonaws.com/i/7sluvf9dajw7mhm5v72c.gif)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
  labels:
    app: hello
spec:
  containers:
  - name: hello-world-container
    image: busybox:1.36
    command: ['sh', '-c', 'echo Hello World! && sleep 3600']
  initContainers:
  - name: mysql-container
    image: busybox:1.36
    command:
    - /bin/sh
    - -c
    - |
      until nc -z -w 1 hello-mysql 3306; do
        echo "DB is not yet reachable; sleeping 10s before retry"
        sleep 10
      done
      echo "DB is UP"
```

The `initContainers` is similar to `containers`. Here we used another `busybox` container and wait till the `hello-mysql` pod is up and running. All `initContainers` must succeed before the main application containers are started. This file exists in this folder as `pod.yaml`.

The `hello-mysql` app looks like this — this is the corrected version of this manifest (see `deployment.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-mysql
  labels:
    app: hello-mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-mysql
  template:
    metadata:
      labels:
        app: hello-mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "changeme"        # use a Secret in production!
        - name: MYSQL_DATABASE
          value: "some-app"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
      volumes:
      - name: data
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: hello-mysql
spec:
  selector:
    app: hello-mysql
  ports:
  - port: 3306
    targetPort: 3306
```

> ⚠️ A few things that were wrong in older versions of this example, and that people commonly get wrong:
> * a `Deployment` needs `apiVersion: apps/v1` **plus** `spec.selector` + `spec.template` — without them the manifest is rejected
> * `emptyDir` is camelCase (`emptydir` is not a valid volume type)
> * the `mysql` image refuses `MYSQL_USER=root` — use `MYSQL_ROOT_PASSWORD` instead
> * the Deployment's pod labels must match the Service `selector`, otherwise the DNS name `hello-mysql` resolves to nothing

Once the `DB is UP`, the application containers are started.

More about `initContainers` [here](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/).

---

## Pods life cycle

Each pod has a `status` object. The `status` object holds the `phase` field.

> The phase is not intended to be a comprehensive rollup of observations of Container or Pod state, nor is it intended to be a comprehensive state machine.

[![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--GUaCvSLY--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://thepracticaldev.s3.amazonaws.com/i/3kfhr014pwme691k7yqm.jpeg)](https://res.cloudinary.com/practicaldev/image/fetch/s--GUaCvSLY--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://thepracticaldev.s3.amazonaws.com/i/3kfhr014pwme691k7yqm.jpeg)

The lifecycle in the correct order:

1. The controller (e.g. a Deployment) creates the Pod and the **scheduler** assigns it to a node. Until that happens the Pod is in the `Pending` phase.

2. The **kubelet** on the chosen node pulls the container image(s) and creates the containers. The Pod stays `Pending` while images are being downloaded.

3. Once at least one container is running (or starting/restarting), the Pod is marked `Running`. Note: `Running` does not guarantee the application inside is actually ready to serve — that is what probes are for (see below).

4. Eventually all containers terminate — either successfully (`Succeeded`) or with a failure (`Failed`).

Sometimes, the Kubernetes control plane will not be able to connect with the node running the pod (e.g. node down). Pods on it are marked with `Unknown` phase.

> 💡 The main advantage of Kubernetes is its ability to restart pods/containers automagically. We can define this behaviour per pod with the `restartPolicy` field. It accepts `Always | OnFailure | Never` (note the casing — it is `OnFailure`, not `onFailure`). The default value is `Always`.

| restartPolicy | Restarts when |
|---|---|
| `Always` (default) | container exits, whatever the exit code |
| `OnFailure` | container exits with a non-zero code |
| `Never` | never restarts |

---

## Probe it

When running containers, the container might be running but Kubernetes cannot confirm whether the application works correctly. For that, we need to check periodically whether the endpoint or the application in the container is responding correctly. Kubernetes provides three types of handlers, that will probe the container and ensure that it is in the correct and expected state.

[![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--AqpvGP-r--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://thepracticaldev.s3.amazonaws.com/i/0qnfowu4ri1vj1o0fq6j.jpeg)](https://res.cloudinary.com/practicaldev/image/fetch/s--AqpvGP-r--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://thepracticaldev.s3.amazonaws.com/i/0qnfowu4ri1vj1o0fq6j.jpeg)

**The handlers are**:

1. ExecAction

2. TCPSocketAction

3. HTTPGetAction


They are simple to understand. They allow us to probe by executing a command inside the container, check for TCP connection or HTTP request respectively.

The probe can result in either `Success` | `Failure` | `Unknown`.

But it is important to understand when to probe. For example, in the previous example with the Database, we were waiting until the Database is UP. During this time any probe to check the application is not possible. So Kubernetes has three kinds of probes for running containers:

1. **startupProbe** — runs first and disables the other probes until it succeeds. Perfect for slow-starting apps: instead of a giant `initialDelaySeconds`, give the app a startupProbe with a generous `failureThreshold × periodSeconds` budget.

2. **readinessProbe** — probes the container until it is ready to serve traffic. A failing readiness probe *does not* restart the container — it only removes the Pod from the Service endpoints, so no traffic is routed to it. The pod is made available to Services only when the `readinessProbe` succeeds.

3. **livenessProbe** — indicates whether the container is `alive`. When the probe fails the container is killed and then restarted based on the `restartPolicy`.

Consider that your application has a `health` endpoint. That tells the application's health.

```yaml
readinessProbe:
   httpGet:
     path: /healthz
     port: http           # a *named* container port, declared under spec.containers[].ports
   initialDelaySeconds: 5
   periodSeconds: 15
   failureThreshold: 6
```

The above readinessProbe does an `HTTPGetAction` at the path specified `/healthz`. It initially waits 5 seconds to start the probing (`initialDelaySeconds`). Then for every 15 seconds (`periodSeconds`) it probes the container for readiness.

The `failureThreshold` here specifies mark the probe as `failure` only after receiving 6 consecutive failure results.

> ⚠️ `port: http` only works if the container declares a named port:
> ```yaml
> ports:
> - name: http
>   containerPort: 8080
> ```
> Otherwise use a number (`port: 8080`).

```yaml
livenessProbe:
 httpGet:
   path: /healthz
   port: http
 initialDelaySeconds: 20
```

Here the livenessProbe checks the same endpoint as above and starts checking after an initial delay of 20 seconds. If the app takes a long time to boot, prefer a `startupProbe` over a huge `initialDelaySeconds` — otherwise Kubernetes may restart the app before it finishes starting (a crash loop).

Both complete, runnable examples are in this folder: `livenessprobe.yaml` and `readinessprobe.yaml`.

---

## Limitations

The pods has containers inside them. It is very important for us to limit the amount of `memory` and `cpus` that it can utilize.

> CPU and memory are collectively referred to as compute resources, or just resources.

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1"
```

Here we are specifying the container is guaranteed `512Mi` of memory and `500m` CPU — that is **half of one CPU core** (1000m = 1 full core; this is not a percentage of the whole node).

The `limits` specify the maximum memory and CPU allowed to the container. When the container exceeds its memory limit it is killed with `OOMKilled` and restarted according to the `restartPolicy`. Exceeding the CPU limit only throttles the container — it is not restarted.

It is very important to give proper values here and allocate memory and CPU efficiently.

> Resources are defined per container (and the Pod's totals are the sum of its containers).

---

## Try it yourself 🧪

```bash
# from this directory
kubectl apply -f deployment.yaml   # MySQL + Service
kubectl get pods -w                # watch it start

kubectl apply -f pod.yaml          # app pod with initContainer waiting for the DB
kubectl logs hello-world -c mysql-container -f   # watch the init container wait, then succeed

kubectl delete -f pod.yaml -f deployment.yaml
```

---

Hopefully, this might have given you a brief overview of `Pods` in Kubernetes. Head over [kubernetes.io](https://kubernetes.io/) for more information on Kubernetes.

Want to understand the whole picture of Kubernetes and how it works check out [this](https://harshhaa.hashnode.dev/kubernetes-for-everyone) post.

### If you like this article, please share with others. ❤️
