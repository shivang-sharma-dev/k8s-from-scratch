# StatefulSets

## What is a StatefulSet?

A StatefulSet is for **stateful applications** that need stable identity and persistent storage. Unlike Deployments where pods are interchangeable, StatefulSet pods are unique.

## StatefulSet vs Deployment

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| **Pod names** | Random: `nginx-abc123-xyz` | Ordered: `mysql-0`, `mysql-1`, `mysql-2` |
| **Startup order** | All pods start simultaneously | Sequential: 0 → 1 → 2 |
| **Shutdown order** | Random | Reverse: 2 → 1 → 0 |
| **Storage** | Shared or none | Each pod gets its OWN PVC |
| **DNS** | Pods are not individually addressable | Each pod gets a stable DNS name |
| **Use case** | Stateless apps (APIs, frontends) | Databases, Kafka, Zookeeper, Redis |

## Why Ordered Startup Matters

For databases like MySQL with primary/replica architecture:
1. `mysql-0` starts first → becomes the PRIMARY
2. `mysql-1` starts second → clones data from `mysql-0`
3. `mysql-2` starts third → clones data from `mysql-0`

If they all started at once, there'd be no primary to clone from.

## Stable Network Identity

Each StatefulSet pod gets a predictable DNS name:

```
<pod-name>.<headless-service>.<namespace>.svc.cluster.local
```

Example with a StatefulSet named `mysql` and headless service `mysql`:
```
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
mysql-2.mysql.default.svc.cluster.local
```

These DNS names **persist** even when the pod is deleted and recreated — the new pod gets the exact same name and DNS entry.

## Headless Service (Required)

A StatefulSet requires a **headless Service** (a Service with `clusterIP: None`). This creates DNS records for each individual pod instead of a single virtual IP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None          # THIS makes it headless
  selector:
    app: mysql
  ports:
    - port: 3306
```

## Persistent Storage

Each pod gets its own PVC via `volumeClaimTemplates`. When a pod is deleted and recreated, it **reattaches to the same volume** — data is preserved.

```yaml
volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

This creates PVCs named `data-mysql-0`, `data-mysql-1`, `data-mysql-2`. Each pod exclusively owns its PVC.

> **Important:** Deleting a StatefulSet does NOT delete its PVCs. Data is preserved. You must manually delete PVCs if you want to clean up storage.

## When to Use StatefulSets

✅ Use StatefulSet when your app needs:
- Stable, unique network identifiers
- Stable, persistent storage
- Ordered deployment and scaling
- Ordered, graceful shutdown

Common examples: MySQL, PostgreSQL, MongoDB, Kafka, Elasticsearch, Zookeeper, Redis Cluster

❌ Don't use for: stateless apps (use Deployments instead)

## Key Commands

```bash
kubectl get statefulsets -n <ns>
kubectl describe statefulset <name>
kubectl scale statefulset <name> --replicas=5
kubectl rollout status statefulset/<name>
kubectl get pvc                               # See the individual PVCs created
```
