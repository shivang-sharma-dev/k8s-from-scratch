# 02 — Workloads

Covers higher-level abstractions for running containers reliably.

## Topics
- **Deployments** — rolling updates, rollbacks, declarative pod management
- **ReplicaSets** — ensuring a desired number of pod replicas
- **DaemonSets** — run exactly one pod on every node
- **StatefulSets** — ordered, stable identity for stateful applications
- **Jobs** — run-to-completion batch tasks
- **CronJobs** — scheduled recurring jobs

## Files
| File | Description |
|------|-------------|
| `deployment.yml` | Basic Deployment with replicas |
| `replicasets.yml` | Standalone ReplicaSet |
| `daemonsets.yml` | DaemonSet running on every node |
