# Deployments

## What is a Deployment?

A Deployment is the **most common way to run applications** on Kubernetes. It manages a set of identical pods, ensuring the desired number are always running, and provides:

- **Self-healing** — replaces crashed/deleted pods automatically
- **Rolling updates** — update pods one by one with zero downtime
- **Rollback** — revert to any previous version instantly
- **Scaling** — increase or decrease replica count on demand

## How Deployments Work

```
You create → Deployment → creates → ReplicaSet → creates → Pods
```

A Deployment never creates pods directly. It creates a **ReplicaSet**, and the ReplicaSet creates and manages the pods. When you update a Deployment, it creates a NEW ReplicaSet and gradually shifts traffic.

```
Deployment: nginx-deployment
│
├── ReplicaSet: nginx-deployment-abc123 (old - 0 replicas)
│
└── ReplicaSet: nginx-deployment-def456 (current - 3 replicas)
    ├── Pod: nginx-deployment-def456-xxxxx
    ├── Pod: nginx-deployment-def456-yyyyy
    └── Pod: nginx-deployment-def456-zzzzz
```

## Update Strategies

### RollingUpdate (Default)

Replaces pods gradually — new pods come up before old ones are removed.

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # How many EXTRA pods can exist during update
      maxUnavailable: 0    # How many pods can be DOWN during update
```

| Setting | What It Means |
|---------|--------------|
| `maxSurge: 1` | During update, allow 4 pods temporarily (3 desired + 1 extra) |
| `maxUnavailable: 0` | Never go below 3 running pods during update |
| `maxSurge: 25%` | Allow 25% more pods than desired count |

**Best for:** Production workloads where you need zero downtime.

### Recreate

Kills ALL old pods first, then creates new ones. There IS downtime.

```yaml
spec:
  strategy:
    type: Recreate
```

**Best for:** Dev environments or apps that can't run two versions simultaneously (e.g., apps that lock a database).

## Rolling Update In Action

When you change the image from `nginx:1.24` to `nginx:1.25`:

```
Step 1: [v1] [v1] [v1]              ← 3 old pods running
Step 2: [v1] [v1] [v1] [v2]         ← 1 new pod starting (maxSurge=1)
Step 3: [v1] [v1] [v2]              ← 1 old terminated after new is ready
Step 4: [v1] [v1] [v2] [v2]         ← another new pod starting
Step 5: [v1] [v2] [v2]              ← another old terminated
Step 6: [v1] [v2] [v2] [v2]         ← last new pod starting
Step 7: [v2] [v2] [v2]              ← done, all pods updated
```

## Rollback

Every update creates a new "revision." You can roll back anytime:

```bash
# See rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to a specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Check rollout status
kubectl rollout status deployment/nginx-deployment
```

## Scaling

```bash
# Scale manually
kubectl scale deployment nginx-deployment --replicas=5

# Or edit the YAML and re-apply
# replicas: 5
kubectl apply -f deployment.yml
```

For automatic scaling based on CPU/memory, use a **HorizontalPodAutoscaler (HPA)** — covered in 08-observability.

## Deployment vs ReplicaSet vs Pod

| Resource | Manages | Has Updates? | Self-heals? | Use When |
|----------|---------|-------------|-------------|----------|
| **Pod** | Just itself | ❌ | ❌ | Never in prod — learning only |
| **ReplicaSet** | N identical pods | ❌ | ✅ | Almost never directly |
| **Deployment** | ReplicaSets → Pods | ✅ Rolling updates + rollback | ✅ | Always use this for stateless apps |

## Key Commands

```bash
kubectl get deployments -n <ns>
kubectl describe deployment <name>
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl scale deployment <name> --replicas=5
kubectl set image deployment/<name> nginx=nginx:1.25
kubectl rollout restart deployment/<name>         # Restart all pods gracefully

# Watch pods cycling during an update
kubectl get pods -w
```
