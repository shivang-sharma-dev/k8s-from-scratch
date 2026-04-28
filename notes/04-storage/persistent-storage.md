# Persistent Storage — PV, PVC, StorageClasses

## The Problem

Containers are ephemeral — when a pod dies, all data inside it is **lost**. For stateful apps (databases, file uploads, logs), you need storage that survives pod restarts.

## How K8s Storage Works

```
StorageClass (defines HOW to provision storage)
    ↓ dynamically creates
PersistentVolume — PV (a piece of actual storage — disk, NFS, cloud volume)
    ↓ bound to
PersistentVolumeClaim — PVC (a pod's request for storage)
    ↓ mounted into
Pod → Container (via volumeMounts)
```

---

## Volumes (Basic)

A basic Volume is attached to a pod's lifecycle — it survives container restarts but NOT pod deletion.

### emptyDir
Created when a pod starts, deleted when the pod is removed. Good for temp files or sharing data between containers in the same pod.

```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: temp
          mountPath: /tmp/data
  volumes:
    - name: temp
      emptyDir: {}
```

### hostPath
Mounts a file or directory from the host node's filesystem. Data persists on that specific node.

```yaml
volumes:
  - name: host-data
    hostPath:
      path: /var/log
      type: Directory
```

> ⚠️ **Warning:** hostPath is node-specific. If the pod moves to a different node, it won't find the data. Avoid in production.

---

## PersistentVolume (PV)

A PV is a **piece of storage** provisioned in the cluster. It's a cluster-level resource (not namespaced) that represents actual disk — could be a local disk, NFS share, AWS EBS volume, etc.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

### Access Modes

| Mode | Short | Meaning |
|------|-------|---------|
| `ReadWriteOnce` | RWO | Can be mounted as read-write by a **single node** |
| `ReadOnlyMany` | ROX | Can be mounted as read-only by **many nodes** |
| `ReadWriteMany` | RWX | Can be mounted as read-write by **many nodes** |

> Note: Not all storage backends support all modes. Cloud block storage (EBS, GCE PD) typically only supports RWO.

### Reclaim Policies

What happens to the PV after the PVC is deleted?

| Policy | Behavior |
|--------|----------|
| `Retain` | PV stays with data intact. Admin must manually clean up. |
| `Delete` | PV and underlying storage are deleted automatically. |
| `Recycle` | (Deprecated) Runs `rm -rf` on the volume and makes it available again. |

---

## PersistentVolumeClaim (PVC)

A PVC is a **request for storage** by a pod. K8s finds a matching PV (or dynamically creates one) and binds them together.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: dev-env
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi          # Request 5 GiB (must be ≤ PV capacity)
```

### Using PVC in a Pod

```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: data
          mountPath: /var/data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: my-pvc
```

### PV-PVC Binding

K8s matches a PVC to a PV based on:
1. **Capacity** — PV must have ≥ requested storage
2. **Access mode** — Must match
3. **StorageClass** — Must match (or both be empty)
4. **Labels/selectors** — If specified

---

## StorageClass (Dynamic Provisioning)

Instead of manually creating PVs, a StorageClass automatically provisions them when a PVC is created.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs     # Cloud provider plugin
parameters:
  type: gp3                            # EBS volume type
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

### Using StorageClass in a PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fast-storage
spec:
  storageClassName: fast-ssd        # References the StorageClass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

Now when this PVC is created, K8s automatically provisions a 20Gi EBS gp3 volume — no manual PV needed.

## Summary

| Resource | Scope | Who Creates It | Purpose |
|----------|-------|---------------|---------|
| **StorageClass** | Cluster | Admin (once) | Defines HOW to provision storage |
| **PersistentVolume (PV)** | Cluster | Admin or auto (via StorageClass) | A piece of actual storage |
| **PersistentVolumeClaim (PVC)** | Namespace | Developer | A pod's request for storage |

## Key Commands

```bash
kubectl get pv                        # List PersistentVolumes
kubectl get pvc -n <ns>               # List PersistentVolumeClaims
kubectl get storageclass              # List StorageClasses
kubectl describe pv <name>
kubectl describe pvc <name>
kubectl delete pvc <name>             # Unbinds the PV (reclaim policy decides what happens)
```
