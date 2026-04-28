# Security — RBAC, ServiceAccounts, PodSecurity

## RBAC (Role-Based Access Control)

RBAC controls **who can do what** in your cluster. It answers: "Can user/service X perform action Y on resource Z?"

### RBAC Building Blocks

```
WHO (Subject)        +     WHAT (Role)          =     Binding
─────────────              ──────────                  ───────
User                       Role                       RoleBinding
Group                      ClusterRole                ClusterRoleBinding
ServiceAccount
```

### Role vs ClusterRole

| | Role | ClusterRole |
|---|------|------------|
| **Scope** | Single namespace | Entire cluster |
| **Use case** | "Can read pods in `dev` namespace" | "Can read pods in ALL namespaces" |
| **Bound by** | RoleBinding | ClusterRoleBinding |

### Creating a Role

"Allow reading pods and logs in the `dev` namespace":

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
  - apiGroups: [""]               # "" = core API group (pods, services, etc.)
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
```

### Available Verbs

| Verb | Meaning |
|------|---------|
| `get` | Read a single resource |
| `list` | List all resources |
| `watch` | Stream changes in real-time |
| `create` | Create new resources |
| `update` | Modify existing resources |
| `patch` | Partially modify resources |
| `delete` | Delete resources |
| `*` | All verbs |

### Binding a Role to a ServiceAccount

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: dev
subjects:
  - kind: ServiceAccount
    name: app-sa
    namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRole + ClusterRoleBinding (Cluster-wide)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-viewer-binding
subjects:
  - kind: ServiceAccount
    name: monitoring-sa
    namespace: monitoring
roleRef:
  kind: ClusterRole
  name: node-viewer
  apiGroup: rbac.authorization.k8s.io
```

---

## ServiceAccounts

A ServiceAccount is an **identity for pods** to authenticate against the Kubernetes API.

Every pod runs with a ServiceAccount. If you don't specify one, it uses `default`.

### Why Custom ServiceAccounts?

The `default` SA has minimal permissions. If your app needs to interact with the K8s API (list pods, create jobs, etc.), create a dedicated SA with specific permissions.

### Creating a ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: dev
```

### Using in a Pod

```yaml
spec:
  serviceAccountName: app-sa
  automountServiceAccountToken: true      # Default is true
```

### Disabling Auto-Mount (Security Hardening)

If your pod doesn't need K8s API access, disable the token mount:

```yaml
spec:
  serviceAccountName: app-sa
  automountServiceAccountToken: false
```

---

## Pod Security

### Pod Security Standards (PSS)

K8s defines 3 security levels that can be enforced per namespace:

| Level | What it allows |
|-------|---------------|
| **Privileged** | No restrictions. Full access. |
| **Baseline** | Blocks known privilege escalations. Allows most workloads. |
| **Restricted** | Heavily locked down. Best security practices. |

### Enforcement Modes

| Mode | Behavior |
|------|----------|
| `enforce` | Blocks pods that violate the policy |
| `audit` | Allows but logs violations |
| `warn` | Allows but shows a warning to the user |

### Applying PSS to a Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Security Context (Pod-Level)

Fine-grained security settings for individual pods:

```yaml
spec:
  securityContext:
    runAsUser: 1000              # Don't run as root
    runAsGroup: 3000
    fsGroup: 2000
    runAsNonRoot: true           # Fail if image tries to run as root
  containers:
    - name: app
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true       # Container filesystem is read-only
        capabilities:
          drop: ["ALL"]                    # Drop all Linux capabilities
```

## Key Commands

```bash
# RBAC
kubectl get roles -n <ns>
kubectl get rolebindings -n <ns>
kubectl get clusterroles
kubectl get clusterrolebindings
kubectl auth can-i get pods --as system:serviceaccount:dev:app-sa    # Test permissions

# ServiceAccounts
kubectl get sa -n <ns>
kubectl describe sa <name>
```
