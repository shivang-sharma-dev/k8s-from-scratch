# ☸️ Kubernetes Daily Cheatsheet — All-in-One Reference

> **Pro Tip:** Always specify `-n <namespace>` explicitly to avoid operating on the wrong namespace. Default namespace is `default`.

---

## 📌 Quick Namespace Trick

```bash
# Set a default namespace for your session (avoid typing -n every time)
kubectl config set-context --current --namespace=<namespace>

# Verify current context and namespace
kubectl config get-contexts
kubectl config current-context
```

---

## 🔍 1. GET / INSPECT Resources

```bash
# List resources
kubectl get pods                          # Pods in current namespace
kubectl get pods -n <namespace>           # Pods in specific namespace
kubectl get pods -A                       # Pods across ALL namespaces
kubectl get pods -o wide                  # With node & IP info
kubectl get pods --show-labels            # Show labels
kubectl get pods -l app=nginx             # Filter by label

kubectl get all -n <namespace>            # All resources in namespace
kubectl get all -A                        # All resources cluster-wide

kubectl get nodes                         # Cluster nodes
kubectl get nodes -o wide                 # With OS, IP, kernel info

kubectl get namespaces                    # List all namespaces
kubectl get deployments -n <namespace>
kubectl get services -n <namespace>
kubectl get ingress -n <namespace>
kubectl get configmaps -n <namespace>
kubectl get secrets -n <namespace>
kubectl get pvc -n <namespace>            # PersistentVolumeClaims
kubectl get pv                            # PersistentVolumes (cluster-scoped)
kubectl get events -n <namespace>         # Recent events (great for debugging!)
kubectl get events --sort-by='.lastTimestamp' -n <namespace>

# Watch resources live (auto-refreshes)
kubectl get pods -n <namespace> -w
kubectl get pods -A -w
```

---

## 🔬 2. DESCRIBE — Deep Dive into Resources

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl describe deployment <deploy-name> -n <namespace>
kubectl describe service <svc-name> -n <namespace>
kubectl describe node <node-name>
kubectl describe ingress <ingress-name> -n <namespace>
kubectl describe pvc <pvc-name> -n <namespace>

# 💡 Tip: describe shows Events at the bottom — critical for debugging crashes
```

---

## 📄 3. APPLY / CREATE — Manifest Management

```bash
# Apply a manifest (creates or updates — IDEMPOTENT, prefer this)
kubectl apply -f <file.yaml>
kubectl apply -f <directory/>             # Apply all YAMLs in a folder
kubectl apply -f .                        # Apply all YAMLs in current dir
kubectl apply -f <url>                    # Apply from a URL

# Create (fails if resource already exists)
kubectl create -f <file.yaml>

# Dry-run — validate without applying
kubectl apply -f <file.yaml> --dry-run=client
kubectl apply -f <file.yaml> --dry-run=server   # More accurate (hits API server)

# Generate a YAML manifest without creating (great for boilerplate)
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl create service clusterip my-svc --tcp=80:8080 --dry-run=client -o yaml
kubectl create configmap my-config --from-literal=key=value --dry-run=client -o yaml
```

---

## ❌ 4. DELETE Resources

```bash
# Delete by resource type and name — ALWAYS specify namespace!
kubectl delete pod <pod-name> -n <namespace>
kubectl delete deployment <deploy-name> -n <namespace>
kubectl delete service <svc-name> -n <namespace>
kubectl delete ingress <ingress-name> -n <namespace>
kubectl delete configmap <cm-name> -n <namespace>
kubectl delete secret <secret-name> -n <namespace>
kubectl delete pvc <pvc-name> -n <namespace>

# Delete from manifest file
kubectl delete -f <file.yaml>
kubectl delete -f <directory/>

# Delete all pods of a type (deployment manages replacement automatically)
kubectl delete pods -l app=nginx -n <namespace>

# Force delete a stuck pod
kubectl delete pod <pod-name> -n <namespace> --grace-period=0 --force

# Delete ALL resources in a namespace (DANGER!)
kubectl delete all --all -n <namespace>

# ⚠️ YOUR ISSUE: You ran `kubectl get pods -n nginx-app` but deleted without `-n nginx-app`
# CORRECT command for your case:
kubectl delete deployment nginx-deployment -n nginx-app
```

---

## 📝 5. EDIT Resources Live

```bash
# Opens resource in your $EDITOR (vim/nano)
kubectl edit deployment <deploy-name> -n <namespace>
kubectl edit service <svc-name> -n <namespace>
kubectl edit configmap <cm-name> -n <namespace>

# Patch a resource inline (great for scripts)
kubectl patch deployment <deploy-name> -n <namespace> \
  -p '{"spec":{"replicas":3}}'

# Update image of a container in a deployment
kubectl set image deployment/<deploy-name> \
  <container-name>=<new-image>:<tag> -n <namespace>
# Example:
kubectl set image deployment/django-app django=shivangsharma/notes-app:v2 -n my-app
```

---

## 📈 6. SCALE Deployments

```bash
# Scale up/down imperatively
kubectl scale deployment <deploy-name> --replicas=3 -n <namespace>
kubectl scale deployment <deploy-name> --replicas=0 -n <namespace>   # "Pause" app

# Scale multiple deployments at once
kubectl scale deployment deploy1 deploy2 --replicas=2 -n <namespace>

# Autoscale (HPA)
kubectl autoscale deployment <deploy-name> \
  --min=2 --max=10 --cpu-percent=70 -n <namespace>
kubectl get hpa -n <namespace>
```

---

## 🚀 7. ROLLOUT — Deployments & Rollbacks

```bash
# View rollout status
kubectl rollout status deployment/<deploy-name> -n <namespace>

# View rollout history
kubectl rollout history deployment/<deploy-name> -n <namespace>

# Rollback to previous version
kubectl rollout undo deployment/<deploy-name> -n <namespace>

# Rollback to a specific revision
kubectl rollout undo deployment/<deploy-name> --to-revision=2 -n <namespace>

# Pause/resume a rollout
kubectl rollout pause deployment/<deploy-name> -n <namespace>
kubectl rollout resume deployment/<deploy-name> -n <namespace>

# Restart all pods in a deployment (triggers rolling restart)
kubectl rollout restart deployment/<deploy-name> -n <namespace>
```

---

## 🔗 8. LOGS — Debugging

```bash
# View pod logs
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --tail=100       # Last 100 lines
kubectl logs <pod-name> -n <namespace> -f               # Follow/stream logs
kubectl logs <pod-name> -n <namespace> --since=1h       # Logs from last 1 hour

# Multi-container pod — specify container
kubectl logs <pod-name> -c <container-name> -n <namespace>

# Logs from previous (crashed) container instance
kubectl logs <pod-name> -n <namespace> --previous

# Logs from ALL pods matching a label
kubectl logs -l app=django -n <namespace> --tail=50

# 💡 Combine with grep
kubectl logs <pod-name> -n <namespace> | grep ERROR
```

---

## 💻 9. EXEC — Shell into Pods

```bash
# Interactive shell
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh   # For alpine images

# Run a single command
kubectl exec <pod-name> -n <namespace> -- ls /app
kubectl exec <pod-name> -n <namespace> -- env           # View env vars
kubectl exec <pod-name> -n <namespace> -- cat /etc/config/settings.py

# Multi-container pod
kubectl exec -it <pod-name> -c <container-name> -n <namespace> -- /bin/bash
```

---

## 🌐 10. PORT-FORWARD — Local Access

```bash
# Forward local port to pod port
kubectl port-forward pod/<pod-name> 8080:8000 -n <namespace>

# Forward to a service (recommended — works with replicas)
kubectl port-forward service/<svc-name> 8080:80 -n <namespace>

# Forward to a deployment
kubectl port-forward deployment/<deploy-name> 8080:8000 -n <namespace>

# Bind to all interfaces (accessible from network)
kubectl port-forward service/<svc-name> 8080:80 --address=0.0.0.0 -n <namespace>

# 💡 Your Django app port-forward:
kubectl port-forward service/django-service 8000:8000 -n django-notes-app
```

---

## 🗂️ 11. CONFIGMAPS & SECRETS

```bash
# Create ConfigMap
kubectl create configmap my-config \
  --from-literal=DB_HOST=localhost \
  --from-literal=DB_PORT=5432 \
  -n <namespace>

kubectl create configmap my-config --from-file=config.properties -n <namespace>

# Create Secret
kubectl create secret generic my-secret \
  --from-literal=DB_PASSWORD=supersecret \
  -n <namespace>

kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=<user> \
  --docker-password=<password> \
  -n <namespace>

# Decode a secret value
kubectl get secret <secret-name> -n <namespace> -o jsonpath='{.data.DB_PASSWORD}' | base64 -d

# View ConfigMap data
kubectl get configmap <cm-name> -n <namespace> -o yaml
```

---

## 💾 12. PERSISTENT VOLUMES

```bash
kubectl get pv                                          # Cluster-wide
kubectl get pvc -n <namespace>
kubectl describe pvc <pvc-name> -n <namespace>

# Check storage classes available
kubectl get storageclass
```

---

## 🏷️ 13. LABELS & SELECTORS

```bash
# Add a label to a resource
kubectl label pod <pod-name> env=production -n <namespace>
kubectl label node <node-name> disktype=ssd

# Remove a label
kubectl label pod <pod-name> env- -n <namespace>

# Filter by label
kubectl get pods -l app=nginx,env=prod -n <namespace>
kubectl get pods -l 'env in (prod, staging)' -n <namespace>

# Annotate
kubectl annotate pod <pod-name> description="main app pod" -n <namespace>
```

---

## 🔐 14. RBAC (Role-Based Access Control)

```bash
kubectl get roles -n <namespace>
kubectl get rolebindings -n <namespace>
kubectl get clusterroles
kubectl get clusterrolebindings

kubectl describe role <role-name> -n <namespace>
kubectl describe rolebinding <rb-name> -n <namespace>

# Check what a user/serviceaccount can do
kubectl auth can-i create pods -n <namespace>
kubectl auth can-i create pods --as=system:serviceaccount:<ns>:<sa-name> -n <namespace>

# List all permissions a SA has
kubectl auth can-i --list -n <namespace>
```

---

## 🌍 15. NAMESPACE MANAGEMENT

```bash
kubectl get namespaces
kubectl create namespace <name>
kubectl delete namespace <name>           # ⚠️ Deletes ALL resources in it!

# Get resources across all namespaces
kubectl get pods -A
kubectl get deployments -A
kubectl get services -A
```

---

## 📊 16. RESOURCE USAGE (TOP)

```bash
# Requires metrics-server installed
kubectl top nodes                         # Node CPU/Memory
kubectl top pods -n <namespace>           # Pod CPU/Memory
kubectl top pods -A                       # All namespaces
kubectl top pods --sort-by=cpu -n <namespace>
kubectl top pods --sort-by=memory -n <namespace>
```

---

## 🔧 17. NODE MANAGEMENT

```bash
kubectl get nodes
kubectl describe node <node-name>

# Cordon — prevent new pods from scheduling on node
kubectl cordon <node-name>

# Drain — evict pods and cordon (for maintenance)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Uncordon — re-enable scheduling
kubectl uncordon <node-name>

# Taint a node
kubectl taint nodes <node-name> key=value:NoSchedule
kubectl taint nodes <node-name> key=value:NoSchedule-       # Remove taint
```

---

## 📤 18. OUTPUT FORMATS

```bash
# Output as YAML (export current state)
kubectl get deployment <name> -n <namespace> -o yaml
kubectl get pod <name> -n <namespace> -o yaml > pod-backup.yaml

# Output as JSON
kubectl get pod <name> -n <namespace> -o json

# JSONPath — extract specific fields
kubectl get pod <pod-name> -n <namespace> -o jsonpath='{.status.podIP}'
kubectl get pods -n <namespace> -o jsonpath='{.items[*].metadata.name}'
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Custom columns
kubectl get pods -n <namespace> \
  -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# Wide output
kubectl get pods -o wide -n <namespace>
```

---

## 🛠️ 19. CONTEXT & CLUSTER MANAGEMENT

```bash
kubectl config get-contexts                              # List all contexts
kubectl config current-context                          # Show active context
kubectl config use-context <context-name>               # Switch context

kubectl config set-context --current --namespace=<ns>  # Change default namespace

kubectl cluster-info                                    # Cluster endpoint info
kubectl cluster-info dump                               # Full diagnostic dump
kubectl version                                         # Client & server version
kubectl api-resources                                   # All supported resources
kubectl api-versions                                    # All API versions
```

---

## 🧹 20. USEFUL ONE-LINERS

```bash
# Delete all Evicted pods in a namespace
kubectl get pods -n <namespace> | grep Evicted | awk '{print $1}' | xargs kubectl delete pod -n <namespace>

# Delete all CrashLoopBackOff pods
kubectl get pods -A | grep CrashLoopBackOff | awk '{print $1, $2}' | xargs -L1 bash -c 'kubectl delete pod $2 -n $1'

# Restart all deployments in a namespace
kubectl rollout restart deployment -n <namespace>

# Get all images running in a namespace
kubectl get pods -n <namespace> -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}'

# Copy file from pod to local
kubectl cp <namespace>/<pod-name>:/path/to/file ./local-file

# Copy file from local to pod
kubectl cp ./local-file <namespace>/<pod-name>:/path/in/pod

# Watch events in real time (great for debugging)
kubectl get events -n <namespace> -w --sort-by='.lastTimestamp'

# Apply and watch rollout in one line
kubectl apply -f deploy.yaml && kubectl rollout status deployment/<name> -n <namespace>
```

---

## ⚡ 21. KUBECTL ALIASES (Add to ~/.bashrc or ~/.zshrc)

```bash
alias k='kubectl'
alias kga='kubectl get all -A'
alias kgp='kubectl get pods'
alias kgn='kubectl get nodes'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias kpf='kubectl port-forward'
alias kctx='kubectl config use-context'
alias kns='kubectl config set-context --current --namespace'

# Usage examples:
# k get pods -n django-app
# kaf deploy.yaml
# kns nginx-app       ← sets default namespace
# kctx minikube       ← switch cluster
```

---

## 🏗️ 22. MANIFEST TEMPLATE SNIPPETS

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-namespace
  labels:
    app: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app          # ← Must match template labels
  template:
    metadata:
      labels:
        app: my-app        # ← Must match selector
    spec:
      containers:
        - name: my-app
          image: my-image:latest
          ports:
            - containerPort: 8000
          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: my-config
                  key: DB_HOST
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
  namespace: my-namespace
spec:
  selector:
    app: my-app            # ← Must match Deployment pod labels
  ports:
    - protocol: TCP
      port: 80             # Service port
      targetPort: 8000     # Container port
  type: ClusterIP          # ClusterIP | NodePort | LoadBalancer
```

### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
  namespace: my-namespace
data:
  DB_HOST: "postgres-service"
  DB_PORT: "5432"
  APP_ENV: "production"
```

### Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: my-namespace
type: Opaque
data:
  DB_PASSWORD: c3VwZXJzZWNyZXQ=   # base64 encoded: echo -n 'supersecret' | base64
```

---

## 🔥 Common Mistakes & Fixes

| ❌ Mistake | ✅ Fix |
|---|---|
| Deleting without `-n <namespace>` | Always include `-n <namespace>` |
| Service selector doesn't match pod labels | Ensure `spec.selector` in Service == `metadata.labels` in Deployment template |
| Image pull errors | Check `kubectl describe pod` → Events section |
| Pod stuck in `Pending` | Check node resources: `kubectl describe node`, `kubectl top nodes` |
| Pod `CrashLoopBackOff` | Check logs: `kubectl logs <pod> --previous` |
| `ImagePullBackOff` | Verify image name/tag, check imagePullSecret |
| Port-forward drops | Use `service/` instead of `pod/` — pods are ephemeral |
| Changes not reflected | Run `kubectl rollout restart deployment/<name> -n <ns>` |

---

*Generated for daily Kubernetes cluster management · Last updated: May 2026*
