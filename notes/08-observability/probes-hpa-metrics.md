# Observability — Probes, HPA, Metrics

## Health Probes

Probes let Kubernetes **monitor the health** of your containers and take automatic action when something goes wrong.

### Three Types of Probes

| Probe | Question It Answers | Action on Failure |
|-------|-------------------|-------------------|
| **Liveness** | Is the container alive? | Kill and restart the container |
| **Readiness** | Is the container ready to serve traffic? | Remove from Service endpoints (stop sending traffic) |
| **Startup** | Has the container finished starting? | Kill and restart if startup takes too long |

### Liveness Probe

"Is my container stuck or deadlocked?" If the probe fails, K8s kills the container and restarts it.

```yaml
spec:
  containers:
    - name: app
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 15      # Wait 15s after container starts
        periodSeconds: 10            # Check every 10s
        failureThreshold: 3          # Kill after 3 consecutive failures
```

### Readiness Probe

"Is my container ready to handle requests?" If it fails, the pod is removed from the Service — no traffic is sent to it. The container is NOT restarted.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**Use case:** Your app takes 30 seconds to load data into memory. Without a readiness probe, K8s sends traffic immediately and users get errors.

### Startup Probe

"Has my container finished booting?" Disables liveness and readiness probes until the startup probe succeeds. Designed for slow-starting containers.

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30        # Try 30 times
  periodSeconds: 10            # Every 10s = up to 5 minutes to start
```

### Probe Methods

| Method | How It Works |
|--------|-------------|
| `httpGet` | Makes an HTTP GET request. Success = 200-399 status code. |
| `tcpSocket` | Opens a TCP connection to a port. Success = connection established. |
| `exec` | Runs a command inside the container. Success = exit code 0. |

#### TCP Example (databases)
```yaml
livenessProbe:
  tcpSocket:
    port: 3306
```

#### Exec Example (custom health check)
```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
```

---

## Horizontal Pod Autoscaler (HPA)

HPA automatically scales the number of pod replicas based on CPU usage, memory usage, or custom metrics.

### How It Works

```
HPA watches metrics → CPU > 70%? → Scale up replicas
                    → CPU < 30%? → Scale down replicas
```

### Prerequisites

HPA needs the **Metrics Server** installed to read CPU/memory data:

```bash
# Install metrics server (for kind/minikube)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### HPA YAML

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70       # Scale up when average CPU > 70%
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80       # Scale up when average memory > 80%
```

### Quick CLI Create

```bash
kubectl autoscale deployment api-deployment --min=2 --max=10 --cpu-percent=70
```

### Important Notes

- Your Deployment pods **MUST have resource requests** set for HPA to work (otherwise there's no baseline to calculate utilization percentage).
- Scale-up happens within ~15-30 seconds.
- Scale-down happens slowly (default 5 minute stabilization window) to prevent flapping.

### Testing HPA with Load

```bash
# Generate CPU load to trigger scale-up
kubectl run load-gen --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://api-service; done"

# Watch HPA in action
kubectl get hpa -w
kubectl get pods -w
```

---

## Metrics & Monitoring Commands

```bash
# Node-level resource usage
kubectl top nodes

# Pod-level resource usage
kubectl top pods -n <ns>
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory

# Events (useful for debugging autoscaling decisions)
kubectl get events -n <ns> --sort-by='.lastTimestamp'

# HPA status
kubectl get hpa
kubectl describe hpa <name>
```

## Key Takeaways

| Concept | Purpose |
|---------|---------|
| Liveness probe | Restart crashed/hung containers |
| Readiness probe | Don't send traffic until ready |
| Startup probe | Give slow containers time to boot |
| HPA | Auto-scale replicas based on load |
| Metrics Server | Provides CPU/memory data for HPA and `kubectl top` |
