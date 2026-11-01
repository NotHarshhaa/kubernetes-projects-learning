![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)
# Multi-container Pods (10%)

### Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
vi pod.yaml
```

Copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

```YAML
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
```

```bash
kubectl create -f pod.yaml
# Connect to the busybox2 container within the pod
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit

# or you can do the above with just an one-liner
kubectl exec -it busybox -c busybox2 -- ls

# you can do some cleanup
kubectl delete po busybox
```

</p>
</details>

### Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run box --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

Copy/paste the container related values, so your final YAML should contain the volume and the initContainer:

Volume:

```YAML
containers:
- image: nginx
...
  volumeMounts:
  - name: vol
    mountPath: /usr/share/nginx/html
volumes:
- name: vol
  emptyDir: {}
```

initContainer:

```YAML
...
initContainers:
- args:
  - /bin/sh
  - -c
  - "wget -O /work-dir/index.html http://neverssl.com/online"
  image: busybox
  name: box
  volumeMounts:
  - name: vol
    mountPath: /work-dir
```

In total you get:

```YAML

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: box
  name: box
spec:
  initContainers: 
  - args: 
    - /bin/sh 
    - -c 
    - "wget -O /work-dir/index.html http://neverssl.com/online"
    image: busybox 
    name: box 
    volumeMounts: 
    - name: vol 
      mountPath: /work-dir 
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts: 
    - name: vol 
      mountPath: /usr/share/nginx/html 
  volumes: 
  - name: vol 
    emptyDir: {} 
```

```bash
# Apply pod
kubectl apply -f pod-init.yaml

# Get IP
kubectl get po -o wide

# Execute wget
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- $(kubectl get pod box -o jsonpath='{.status.podIP}')"

# you can do some cleanup
kubectl delete po box
```

</p>
</details>

### Create a Pod with an nginx container and a busybox sidecar container that continuously logs the date to a shared emptyDir volume. Verify the sidecar's output, then delete the Pod

*Note*: there are two "sidecar" patterns. The **classic** one is a second regular container (`spec.containers`). Since Kubernetes 1.29 you can also use a **native sidecar**: an `initContainer` with `restartPolicy: Always` — it starts before (and alongside) the main containers, runs for the Pod's whole lifetime, and doesn't block Pod startup. Both are shown below.

<details><summary>show</summary>
<p>

```bash
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logger
spec:
  # native sidecar (K8s >= 1.29): starts before and runs alongside the main container
  initContainers:
  - name: logger
    image: busybox:1.36
    restartPolicy: Always        # <-- this makes the init container a native sidecar
    command: ['/bin/sh', '-c', 'while true; do date >> /var/log/dates.log; sleep 2; done']
    volumeMounts:
    - name: logs
      mountPath: /var/log
  containers:
  - name: web
    image: nginx
    volumeMounts:
    - name: logs
      mountPath: /var/log
  volumes:
  - name: logs
    emptyDir: {}
```

```bash
kubectl create -f pod.yaml

# verify the sidecar is writing to the shared volume
kubectl exec web-with-logger -c web -- tail /var/log/dates.log

# check container statuses - the sidecar shows as a running init container
kubectl get pod web-with-logger -o jsonpath='{.status.initContainerStatuses[*].name}{"\n"}'

# classic pattern instead: move the logger to spec.containers as a second entry
# (same volume mount, no restartPolicy needed)

kubectl delete po web-with-logger
```

</p>
</details>

