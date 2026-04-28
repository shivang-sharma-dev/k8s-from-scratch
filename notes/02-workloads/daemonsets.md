# DaemonSets

## What is a DaemonSet?

A DaemonSet ensures that **exactly one copy of a pod runs on every node** (or a subset of nodes) in your cluster. When a new node is added, a pod is automatically created on it. When a node is removed, the pod is garbage collected.

## When to Use DaemonSets

DaemonSets are for **node-level services** that need to run everywhere:

| Use Case | Example |
|----------|---------|
| **Log collection** | Fluentd, Filebeat — collect logs from every node |
| **Node monitoring** | Prometheus Node Exporter, Datadog agent |
| **Networking** | CNI plugins (Calico, Weave), kube-proxy |
| **Storage** | GlusterFS, Ceph agents running per node |
| **Security** | Falco (runtime security), antivirus scanners |

## DaemonSet vs Deployment

| | Deployment | DaemonSet |
|---|-----------|-----------|
| **How many pods** | You decide (replicas: N) | One per node (automatic) |
| **Which nodes** | Scheduler picks best nodes | ALL nodes (or filtered subset) |
| **Scaling** | Manual or HPA | Scales with cluster size |
| **Use case** | Application workloads | Infrastructure / node agents |

## Running on Specific Nodes Only

By default, a DaemonSet runs on ALL nodes. Use `nodeSelector` or `affinity` to target specific nodes:

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disk: ssd              # Only run on nodes with this label

      # OR using tolerations to also run on control-plane nodes
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
          effect: NoSchedule
```

> **Note:** By default, DaemonSet pods do NOT run on the control-plane node because it has a taint. Add the toleration above if you want them there too.

## Updating DaemonSets

DaemonSets support rolling updates just like Deployments:

```yaml
spec:
  updateStrategy:
    type: RollingUpdate          # Default
    rollingUpdate:
      maxUnavailable: 1          # Update 1 node at a time
```

| Strategy | Behavior |
|----------|----------|
| `RollingUpdate` (default) | Updates one node at a time |
| `OnDelete` | Only updates when you manually delete old pods |

## Key Commands

```bash
kubectl get daemonsets -n <ns>
kubectl describe daemonset <name>
kubectl rollout status daemonset/<name>
kubectl rollout undo daemonset/<name>

# See which pods are on which nodes
kubectl get pods -n <ns> -o wide -l <label-selector>
```
