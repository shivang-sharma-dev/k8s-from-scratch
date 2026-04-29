# ⚡ kubectl Cheatsheet

## Cluster Info
```bash
kubectl version                          # Client + server version
kubectl cluster-info                     # API server and CoreDNS URLs
kubectl get nodes                        # List all nodes
kubectl get nodes -o wide                # Nodes with IPs and OS info
kubectl describe node <node-name>        # Detailed node info
```

## Pods
```bash
kubectl get pods                         # Pods in default namespace
kubectl get pods -n <namespace>          # Pods in a specific namespace
kubectl get pods -A                      # Pods across ALL namespaces
kubectl get pods -o wide                 # Pods with node and IP info
kubectl describe pod <pod-name>          # Full pod details (events, status)
kubectl logs <pod-name>                  # Container logs
kubectl logs <pod-name> -f               # Follow/stream logs live
kubectl logs <pod-name> -c <container>   # Logs for a specific container
kubectl exec -it <pod-name> -- bash      # Shell into a running pod
kubectl delete pod <pod-name>            # Delete a pod
```

## Apply / Create / Delete
```bash
kubectl apply -f <file.yml>              # Create or update from file
kubectl apply -f ./                      # Apply all YAMLs in a directory
kubectl delete -f <file.yml>             # Delete resources defined in file
kubectl create namespace <name>          # Create a namespace
```

## Deployments
```bash
kubectl get deployments -n <ns>
kubectl describe deployment <name>
kubectl rollout status deployment/<name>          # Watch rollout progress
kubectl rollout history deployment/<name>         # See revision history
kubectl rollout undo deployment/<name>            # Rollback to previous version
kubectl scale deployment <name> --replicas=5      # Scale up/down
kubectl set image deployment/<name> <container>=<image>:<tag>  # Update image
```

## Services
```bash
kubectl get svc -n <namespace>
kubectl describe svc <service-name>
kubectl port-forward svc/<name> 8080:80           # Forward local port to service
```

## ConfigMaps & Secrets
```bash
kubectl get configmaps -n <ns>
kubectl get secrets -n <ns>
kubectl describe secret <name> -n <ns>
kubectl get secret <name> -o jsonpath='{.data.<key>}' | base64 -d   # Decode a secret value
```

## Namespaces
```bash
kubectl get ns                           # List all namespaces
kubectl config set-context --current --namespace=<ns>   # Switch default namespace
```

## Resource Usage
```bash
kubectl top nodes                        # CPU/Memory per node
kubectl top pods -n <ns>                 # CPU/Memory per pod
```

## Events & Debugging
```bash
kubectl get events -n <ns> --sort-by='.lastTimestamp'   # Recent events
kubectl get all -n <ns>                  # All resources in a namespace
kubectl explain pod.spec.containers      # API field docs inline
```

## Useful Flags
| Flag | Meaning |
|------|---------|
| `-n <namespace>` | Target namespace |
| `-A` | All namespaces |
| `-o wide` | Extra columns |
| `-o yaml` | Full YAML output |
| `-o json` | Full JSON output |
| `--dry-run=client -o yaml` | Preview without applying |
| `--watch` / `-w` | Watch for changes |
