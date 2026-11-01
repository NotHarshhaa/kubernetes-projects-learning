![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/observability&empty)
# Observability (18%)

## Liveness, readiness and startup probes

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

### Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe: # our probe
      exec: # add this line
        command: # command definition
        - ls # ls command
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i liveness # run this to see that liveness probe works
kubectl delete -f pod.yaml
```

</p>
</details>

### Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl explain pod.spec.containers.livenessProbe # get the exact names
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe:
      initialDelaySeconds: 5 # add this line
      periodSeconds: 5 # add this line as well
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe po nginx | grep -i liveness
kubectl delete -f pod.yaml
```

</p>
</details>

### Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml --restart=Never --port=80 > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    ports:
      - containerPort: 80 # Note: Readiness probes runs on the container during its whole lifecycle. Since nginx exposes 80, containerPort: 80 is not required for readiness to work.
    readinessProbe: # declare the readiness probe
      httpGet: # add this line
        path: / #
        port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness # to see the pod readiness details
kubectl delete -f pod.yaml
```

</p>
</details>

### Lots of pods are running in `qa`,`alan`,`test`,`production` namespaces.  All of these pods are configured with liveness probe.  Please list all pods whose liveness probe are failed in the format of `<namespace>/<pod name>` per line.

<details><summary>show</summary>
<p>

A typical liveness probe failure event
```
LAST SEEN   TYPE      REASON      OBJECT              MESSAGE
22m         Warning   Unhealthy   pod/liveness-exec   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

collect failed pods namespace by namespace

```sh
kubectl get events -o json | jq -r '.items[] | select(.message | contains("Liveness probe failed")).involvedObject | .namespace + "/" + .name'
```

Note the filter matches the actual event message text: `Liveness probe failed: ...`. If the events have rotated out of the API (they expire after ~1h), re-trigger the failure first.

</p>
</details>

### Create an nginx pod with a startupProbe that runs 'ls' with a failureThreshold of 30 and periodSeconds of 10 (i.e. up to 5 minutes for the app to boot). Confirm the startupProbe replaces liveness/readiness probing until it succeeds

<details><summary>show</summary>
<p>

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    startupProbe: # runs first, and disables liveness/readiness until it succeeds
      exec:
        command: ['ls']
      failureThreshold: 30
      periodSeconds: 10
    livenessProbe: # only starts AFTER the startupProbe has passed
      exec:
        command: ['ls']
```

```bash
kubectl create -f pod.yaml
kubectl describe po nginx | grep -iE "startup|liveness"
# events show "Starting container" -> "Started container" -> probe transitions
kubectl delete po nginx
```

Why this matters: without a startupProbe, a slow-booting app can be killed by its livenessProbe before it finishes starting (crash loop). The startupProbe gives it a grace window — a much better pattern than a huge `initialDelaySeconds`.

</p>
</details>

## Logging

### Create a busybox pod that runs `i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done`. Check its logs

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox -f # follow the logs
```

</p>
</details>

## Debugging

### Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
# show that there's an error
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```

</p>
</details>

### Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- notexist
kubectl logs busybox # will bring nothing! container never started
kubectl describe po busybox # in the events section, you'll see the error
# also...
kubectl get events | grep -i error # you'll see the error here as well
kubectl delete po busybox --force --grace-period=0
```

</p>
</details>


### Get CPU/memory utilization for nodes and pods ([metrics-server](https://github.com/kubernetes-sigs/metrics-server) must be running)

<details><summary>show</summary>
<p>

```bash
kubectl top nodes
kubectl top pods                     # current namespace
kubectl top pods --all-namespaces    # or -A
kubectl top pods --containers        # per-container breakdown
```

</p>
</details>

### Add a debug (ephemeral) container to a running pod, without restarting it

<details><summary>show</summary>
<p>

Ephemeral containers are the exam-friendly way to debug a distroless/stripped image that has no shell:

```bash
kubectl run nginx --image=nginx --restart=Never

# inject a temporary toolbox container into the running pod
kubectl debug -it nginx --image=busybox:1.36 --target=nginx -- sh

# from inside: test connectivity to the app container's localhost
wget -qO- localhost:80

# or, non-interactive: list processes of the app container via a copy of the pod
kubectl debug nginx --image=busybox:1.36 --copy-to=nginx-debug -- sleep 3600
kubectl delete po nginx nginx-debug
```

</p>
</details>

## API deprecations

### Which apiVersion would you use for a CronJob on Kubernetes 1.31? And for Ingress? Where do you check this quickly?

<details><summary>show</summary>
<p>

- **CronJob**: `batch/v1` (batch/v1beta1 was removed in 1.25)
- **Ingress**: `networking.k8s.io/v1` (extensions/v1beta1 was removed in 1.22)

Fastest checks during the exam:

```bash
kubectl api-resources | grep -i cronjob   # shows the API group & version in use
kubectl explain cronjob                   # shows the apiVersion at the top
```

The deprecation policy and migration guides live at kubernetes.io > Reference > [Deprecated API Migration Guide](https://kubernetes.io/docs/reference/using-api/deprecation-guide/) — bookmark-worthy for the "API deprecations" curriculum topic.

</p>
</details>
