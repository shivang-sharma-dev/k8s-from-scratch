# Namespaces

## What is a Namespace?

A Namespace is a **virtual cluster inside your physical cluster**. It provides isolation so that different teams, projects, or environments don't interfere with each other.

Think of it like folders on your computer — the files inside one folder are separate from another.

## Why Namespaces?

| Problem | How Namespaces Help |
|---------|-------------------|
| Two teams both want a service called `api` | Each team uses their own namespace — `team-a/api` and `team-b/api` coexist |
| Want separate dev and prod environments | Create `dev` and `prod` namespaces in the same cluster |
| Want resource limits per team | Apply ResourceQuotas per namespace |
| Need RBAC boundaries | Bind Roles to specific namespaces |

## Default Namespaces

Every cluster comes with these out of the box:

| Namespace | Purpose |
|-----------|---------|
| `default` | Where resources go if you don't specify a namespace |
| `kube-system` | K8s system components (CoreDNS, kube-proxy, scheduler, etc.) |
| `kube-public` | Readable by everyone, rarely used. Contains cluster info. |
| `kube-node-lease` | Node heartbeats — helps control plane detect node failures |

## Creating a Namespace

### Via YAML
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-env
  labels:
    environment: development
```

### Via CLI
```bash
kubectl create namespace dev-env
```

## Using Namespaces

```bash
# Run commands in a specific namespace
kubectl get pods -n dev-env
kubectl apply -f pod.yml -n dev-env

# Switch your default namespace (so you don't have to type -n every time)
kubectl config set-context --current --namespace=dev-env

# Check current namespace
kubectl config view --minify | grep namespace

# List all namespaces
kubectl get ns
```

## Cross-Namespace Communication

Pods in different namespaces CAN communicate, but they need to use the full DNS name:

```
<service-name>.<namespace>.svc.cluster.local
```

**Example:** A pod in `frontend` namespace wants to talk to a service called `api` in `backend` namespace:
```
http://api.backend.svc.cluster.local:8080
```

Within the **same namespace**, just the service name is enough: `http://api:8080`

## Resource Quotas

You can limit how much CPU, memory, and how many objects a namespace can use:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev-env
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
```

## When NOT to Use Namespaces

- Cluster-scoped resources like **Nodes**, **PersistentVolumes**, and **ClusterRoles** are NOT namespaced. They exist across the entire cluster.
- If you only have a tiny cluster with one team, namespaces might be overkill — `default` is fine.

## Key Commands

```bash
kubectl get ns                                    # List all namespaces
kubectl create ns <name>                          # Create a namespace
kubectl delete ns <name>                          # Delete namespace + ALL resources inside it
kubectl get all -n <namespace>                    # All resources in a namespace
kubectl api-resources --namespaced=true           # List all namespaced resource types
kubectl api-resources --namespaced=false          # List all cluster-scoped resource types
```
