# Pods

## What is a Pod?

A Pod is the **smallest deployable unit** in Kubernetes. It is NOT a container — it is a **wrapper around one or more containers** that share:

- **Network namespace** — all containers in a pod share the same IP address and port space. They talk to each other via `localhost`.
- **Storage volumes** — shared volumes can be mounted into multiple containers in the same pod.
- **Lifecycle** — all containers start and stop together.

Think of a pod as a "logical host" for tightly coupled containers.

## Why Pods? Why Not Just Containers?

Kubernetes doesn't manage containers directly. Pods exist because sometimes you need **helper containers** alongside your main app:

| Pattern | Example |
|---------|---------|
| **Sidecar** | Main app + log shipper (Fluentd) |
| **Ambassador** | App + proxy container for outgoing traffic |
| **Init Container** | Runs before main container (e.g., DB migration script) |

## Pod Lifecycle

```
Pending → Running → Succeeded / Failed
```

| Phase | Meaning |
|-------|---------|
| `Pending` | Accepted by the cluster, but one or more containers are not yet running. Could be downloading the image, or waiting for a node. |
| `Running` | At least one container is running or is in the process of starting/restarting. |
| `Succeeded` | All containers terminated successfully (exit code 0). Won't restart. |
| `Failed` | At least one container terminated with a non-zero exit code. |
| `Unknown` | Pod state cannot be determined — usually a node communication error. |

## Single vs Multi-Container Pods

**Rule of thumb:** Use one container per pod unless the containers are tightly coupled and MUST run together.

### Single container (99% of the time)
```yaml
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

### Multi-container (sidecar pattern)
```yaml
spec:
  containers:
    - name: app
      image: myapp:v1
    - name: log-shipper
      image: fluent/fluentd
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app
  volumes:
    - name: shared-logs
      emptyDir: {}
```

Both containers share the same `shared-logs` volume. The app writes logs there, Fluentd reads and ships them.

## Init Containers

Init containers run **before** the main containers start. They run to completion one at a time.

Use cases:
- Wait for a database to be ready before the app starts
- Run a DB migration
- Download config files

```yaml
spec:
  initContainers:
    - name: wait-for-db
      image: busybox
      command: ['sh', '-c', 'until nc -z mysql-service 3306; do sleep 2; done']
  containers:
    - name: app
      image: myapp:v1
```

## Pod Restart Policy

Controls what happens when a container inside the pod exits:

| Policy | Behavior |
|--------|----------|
| `Always` (default) | Always restart, regardless of exit code. Used by Deployments. |
| `OnFailure` | Restart only if exit code is non-zero. Used by Jobs. |
| `Never` | Never restart. Container stays in terminated state. |

```yaml
spec:
  restartPolicy: OnFailure
```

## Key Commands

```bash
kubectl get pods                         # List pods in default namespace
kubectl get pods -n <namespace>          # List pods in specific namespace
kubectl get pods -o wide                 # Show node and IP info
kubectl describe pod <name>              # Full pod details + events
kubectl logs <pod-name>                  # View container logs
kubectl logs <pod-name> -c <container>   # Logs for specific container in multi-container pod
kubectl exec -it <pod-name> -- bash      # Shell into a running container
kubectl delete pod <pod-name>            # Delete a pod
kubectl get pod <name> -o yaml           # See the full YAML spec K8s is using
```

## Important: Pods Are Ephemeral

Pods are **disposable**. If a pod dies, it's gone — Kubernetes does NOT restart standalone pods. That's why you should almost never create pods directly. Instead, use controllers like **Deployments** or **StatefulSets** that automatically replace failed pods.
