# Storage & Persistent Volumes in Kubernetes

Containers are ephemeral: when a container is recreated, everything written inside it is lost. This guide covers how Kubernetes gives workloads durable storage — and how to separate "what storage exists" from "what storage an app needs".

## The four building blocks

| Object | What it is | Who creates it |
|--------|-----------|----------------|
| **Volume** | Storage tied to a Pod's lifetime (`emptyDir`, `hostPath`, ...) | You, in the Pod spec |
| **PersistentVolume (PV)** | A piece of cluster storage with its own lifecycle, independent of any Pod | Admin (static) or provisioner (dynamic) |
| **PersistentVolumeClaim (PVC)** | A user's "request" for storage — size + access mode | You |
| **StorageClass** | A named "profile" that enables **dynamic provisioning** of PVs on demand | Admin |

The workflow to remember:

```
Pod --(mounts)--> PVC --(binds to)--> PV --(backed by)--> disk / NFS / cloud volume
```

## Files in this folder

| File | Purpose |
|------|---------|
| `pv-pvc-demo.yaml` | Static PV (hostPath) + PVC + a Pod consuming the claim |

## Walkthrough

```bash
kubectl apply -f pv-pvc-demo.yaml

# Check the binding — both should show "Bound"
kubectl get pv,pvc
# NAME                        CAPACITY ACCESS MODES RECLAIM POLICY STATUS  CLAIM
# persistentvolume/demo-pv    1Gi      RWO          Retain         Bound   default/demo-pvc
# NAME                            STATUS VOLUME   CAPACITY ACCESS MODES
# persistentvolumeclaim/demo-pvc  Bound  demo-pv  1Gi      RWO

# Write data, destroy the Pod, recreate it — the data survives
kubectl exec pvc-consumer -- sh -c 'echo "I persist" > /usr/share/nginx/html/hello.txt'
kubectl delete pod pvc-consumer
kubectl apply -f pv-pvc-demo.yaml
kubectl exec pvc-consumer -- cat /usr/share/nginx/html/hello.txt
# I persist
```

That is the whole point: the PVC survives Pod deletion, so the new Pod re-mounts the same storage.

## Access modes

| Mode | Meaning | Typical use |
|------|---------|-------------|
| `ReadWriteOnce` (RWO) | One node can mount it read-write | Databases on cloud block storage |
| `ReadOnlyMany` (ROX) | Many nodes, read-only | Shared static content |
| `ReadWriteMany` (RWX) | Many nodes, read-write | NFS, EFS, Azure Files, CephFS |
| `ReadWriteOncePod` (RWOP) | One *Pod*, k8s ≥ 1.22 | Strict single-writer apps |

> ⚠️ Access modes are **capabilities of the storage backend**, not wishes. A cloud block volume (EBS, PD) physically cannot be mounted RWX — requesting that mode will leave your PVC `Pending` forever.

## Lifecycle & reclaim policy

- A PVC binds to a PV matching **class + capacity ≥ request + access modes**. Matching is 1:1 — one PV serves one claim.
- `persistentVolumeReclaimPolicy` decides what happens to the PV after its PVC is deleted:
  - `Retain` — PV keeps data, must be manually cleaned (safest, the static default)
  - `Delete` — PV and the backing cloud volume are removed (the dynamic-provisioning default)
  - `Recycle` — deprecated, don't use
- PVCs can't be shrunk, only expanded (`spec.resources.requests.storage` up, requires `allowVolumeExpansion: true` on the StorageClass).

## Static vs dynamic provisioning

The demo above is **static**: an admin pre-creates the PV, and it only works because `hostPath` needs no infrastructure.

Real clusters use **dynamic provisioning** — you only write a PVC, and the StorageClass creates the PV for you:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  storageClassName: standard        # whatever your cluster provides (GKE/EKS/AKS have defaults)
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 10Gi
```

```bash
kubectl get storageclass           # see the classes available in your cluster
# NAME       PROVISIONER             RECLAIMPOLICY
# standard   k8s.io/minikube-hostpath  Delete      (default)
```

> 💡 If a PVC stays `Pending`: no StorageClass matches (or none is default), the requested access mode isn't supported, or capacity is insufficient. Diagnose with `kubectl describe pvc <name>` — the Events section tells you exactly why.

## StatefulSets + storage

StatefulSets use `volumeClaimTemplates` to get **one PVC per Pod replica** automatically (see [Kubernetes for Everyone](../kubernetes-for-everyone/README.md)). Deleting a StatefulSet does **not** delete the PVCs — a deliberate safety choice you should know before cleaning up.

## Best practices

1. **Never use `hostPath` in production** — it bypasses scheduling and is a container-escape risk.
2. Always set `storageClassName` explicitly instead of relying on the default.
3. For databases, combine PVs with the lessons from [Monitoring](../Monitoring-and-Logging/README.md): monitor disk usage before it fills.
4. Use CSI drivers for cloud storage (`ebs.csi.aws.com`, `pd.csi.storage.gke.io`, ...) — in-tree cloud provisioners are deprecated.

## Additional Resources

- [Persistent Volumes Concepts](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Dynamic Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)
- [CSI Driver List](https://kubernetes-csi.github.io/docs/drivers.html)
