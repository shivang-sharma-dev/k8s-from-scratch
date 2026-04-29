# 🔧 Troubleshooting Cheatsheet

## Pod Not Starting

```bash
# Step 1 — Check pod status and reason
kubectl get pods -n <ns>
# STATUS examples: Pending, CrashLoopBackOff, ImagePullBackOff, Error, OOMKilled

# Step 2 — Read events (most useful for root cause)
kubectl describe pod <pod-name> -n <ns>
# Look at the Events section at the bottom

# Step 3 — Read logs
kubectl logs <pod-name> -n <ns>
kubectl logs <pod-name> -n <ns> --previous   # Logs from last crashed container
```

## Common Status & Their Causes

| Status | Likely Cause | Fix |
|--------|-------------|-----|
| `Pending` | No node has enough CPU/memory, or no matching node | Check `kubectl describe pod` → Events |
| `ImagePullBackOff` | Wrong image name, tag, or no registry auth | Check image name, add imagePullSecret |
| `CrashLoopBackOff` | Container starts then crashes repeatedly | Check `kubectl logs --previous` |
| `OOMKilled` | Container exceeded memory limit | Increase memory limit |
| `Error` | Container exited with non-zero code | Check logs |
| `Terminating` (stuck) | Finalizer blocking deletion | `kubectl delete pod <name> --grace-period=0 --force` |

## Node Issues

```bash
kubectl get nodes                        # Check if nodes are Ready
kubectl describe node <node-name>        # Check conditions (MemoryPressure, DiskPressure)
kubectl top nodes                        # Check resource usage
```

## Service Not Reachable

```bash
# Check if service exists and has the right port
kubectl get svc -n <ns>
kubectl describe svc <svc-name> -n <ns>

# Check if the selector matches actual pod labels
kubectl get pods -n <ns> --show-labels
# The pod labels must match the service's selector

# Test connectivity from inside the cluster
kubectl run test --image=busybox --restart=Never -it --rm -- wget -qO- http://<svc-name>.<ns>.svc.cluster.local
```

## ConfigMap / Secret Not Mounted

```bash
# Confirm the resource exists in the right namespace
kubectl get configmap <name> -n <ns>
kubectl get secret <name> -n <ns>

# Check the pod spec references the right name
kubectl describe pod <pod-name> | grep -A5 "Mounts\|Volumes\|Env"
```

## General Debug Flow

```
1. kubectl get pods        → What is the STATUS?
2. kubectl describe pod    → What do the EVENTS say?
3. kubectl logs            → What does the app say?
4. kubectl exec -- bash    → Can I get inside and test manually?
```
