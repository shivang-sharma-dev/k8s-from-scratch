# Scheduling — Affinity, Taints & Tolerations

## How the Scheduler Works

When you create a pod, the **kube-scheduler** decides which node to place it on:

```
1. Filter   → Eliminate nodes that CAN'T run the pod (not enough CPU, taints, etc.)
2. Score    → Rank remaining nodes by preference (affinity, resource balance, etc.)
3. Bind     → Assign the pod to the highest-scoring node
```

You can influence this process with the mechanisms below.

---

## Node Selectors (Simple)

The simplest way to control pod placement. Match pods to nodes based on labels.

### Step 1: Label a node
```bash
kubectl label node worker-1 disk=ssd
```

### Step 2: Use nodeSelector in the pod
```yaml
spec:
  nodeSelector:
    disk: ssd               # Only schedule on nodes with this label
```

**Limitation:** nodeSelector only supports exact equality. For complex logic (OR, NOT IN), use Node Affinity.

---

## Node Affinity (Advanced Node Selection)

More powerful than nodeSelector — supports operators like In, NotIn, Exists.

### Required (Hard Rule) — Pod won't schedule if no node matches
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: zone
                operator: In
                values: ["us-east-1a", "us-east-1b"]
```

### Preferred (Soft Rule) — Scheduler tries to match, but schedules anyway if it can't
```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 80                    # Higher weight = stronger preference
          preference:
            matchExpressions:
              - key: disk
                operator: In
                values: ["ssd"]
```

### Operators

| Operator | Meaning |
|----------|---------|
| `In` | Label value is one of the listed values |
| `NotIn` | Label value is NOT one of the listed values |
| `Exists` | Label key exists (any value) |
| `DoesNotExist` | Label key does NOT exist |

---

## Taints & Tolerations

**Taints** are applied to **nodes** — they repel pods.
**Tolerations** are applied to **pods** — they allow pods to be scheduled on tainted nodes.

Think of it as: "Nodes say NO, pods can override with a toleration."

### Taint a Node
```bash
kubectl taint nodes worker-1 gpu=true:NoSchedule
```

This means: no pod will be scheduled on `worker-1` unless it tolerates the `gpu=true` taint.

### Tolerate the Taint (in pod spec)
```yaml
spec:
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
```

### Taint Effects

| Effect | Behavior |
|--------|----------|
| `NoSchedule` | New pods without toleration won't be scheduled here. Existing pods are unaffected. |
| `PreferNoSchedule` | Scheduler tries to avoid this node, but will use it if no alternatives. |
| `NoExecute` | Evicts existing pods that don't tolerate the taint AND prevents new ones. |

### Common Use Cases

| Scenario | Taint |
|----------|-------|
| **GPU nodes** — only GPU workloads | `gpu=true:NoSchedule` |
| **Control plane** — don't run user pods | `node-role.kubernetes.io/control-plane:NoSchedule` (default) |
| **Dedicated nodes** — reserved for a specific team | `team=data:NoSchedule` |
| **Node maintenance** — drain and cordon | `NoExecute` |

### Remove a Taint
```bash
kubectl taint nodes worker-1 gpu=true:NoSchedule-     # Trailing minus removes it
```

---

## Pod Affinity & Anti-Affinity

Control pod placement **relative to other pods** (not nodes).

### Pod Affinity — "Place me near these pods"

Run the cache pod on the same node as the api pod:

```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app: api
          topologyKey: kubernetes.io/hostname    # "Same node"
```

### Pod Anti-Affinity — "Place me AWAY from these pods"

Spread replicas across different nodes (for high availability):

```yaml
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchLabels:
                app: frontend
            topologyKey: kubernetes.io/hostname
```

### topologyKey

| topologyKey | Meaning |
|-------------|---------|
| `kubernetes.io/hostname` | Same/different node |
| `topology.kubernetes.io/zone` | Same/different availability zone |
| `topology.kubernetes.io/region` | Same/different region |

---

## Summary — When to Use What

| Need | Tool |
|------|------|
| Run on nodes with `disk=ssd` | `nodeSelector` |
| Run in zone us-east-1a OR us-east-1b | Node Affinity |
| Keep GPU workloads on GPU nodes only | Taints + Tolerations |
| Spread replicas across nodes | Pod Anti-Affinity |
| Co-locate cache with app on same node | Pod Affinity |
