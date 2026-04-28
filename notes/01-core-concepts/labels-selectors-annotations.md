# Labels, Selectors & Annotations

## Labels

Labels are **key/value pairs** attached to K8s objects. They are the primary mechanism for **organizing, grouping, and selecting** resources.

### Adding Labels
```yaml
metadata:
  labels:
    app: nginx
    tier: frontend
    environment: production
    version: v2
```

### Why Labels Matter

Labels are NOT just metadata — they are how Kubernetes **connects resources**:

- A **Service** finds its pods using label selectors
- A **Deployment** manages pods using label selectors
- A **NetworkPolicy** targets pods using label selectors
- You can filter `kubectl` output using label selectors

```
                    ┌─────────────────────────────────┐
                    │    Service (selector: app=nginx) │
                    └──────────┬──────────────────────┘
                               │ matches
                ┌──────────────┼──────────────────┐
                ▼              ▼                  ▼
          ┌──────────┐  ┌──────────┐       ┌──────────┐
          │ Pod      │  │ Pod      │       │ Pod      │
          │ app=nginx│  │ app=nginx│       │ app=nginx│
          └──────────┘  └──────────┘       └──────────┘
```

### Label Naming Rules

- **Key**: up to 63 characters, alphanumeric, `-`, `_`, `.`
- **Value**: up to 63 characters, same rules
- **Prefix** (optional): `company.com/key` — up to 253 characters

## Selectors

Selectors are how you **query** resources by their labels.

### Equality-Based Selectors
```bash
kubectl get pods -l app=nginx                 # Label equals value
kubectl get pods -l app!=redis                # Label does NOT equal value
kubectl get pods -l environment=prod,tier=frontend   # AND (both must match)
```

### Set-Based Selectors
```bash
kubectl get pods -l 'environment in (prod, staging)'    # Value is one of these
kubectl get pods -l 'environment notin (dev)'            # Value is NOT one of these
kubectl get pods -l 'gpu'                                # Label EXISTS (any value)
kubectl get pods -l '!gpu'                               # Label does NOT exist
```

### Selectors in YAML

**In Services and Deployments:**
```yaml
selector:
  matchLabels:
    app: nginx          # Simple equality

# OR with set-based:
selector:
  matchLabels:
    app: nginx
  matchExpressions:
    - key: environment
      operator: In
      values: [prod, staging]
```

## Annotations

Annotations are also key/value pairs, but they are for **non-identifying metadata**. Kubernetes itself does NOT use annotations for selection — they're informational or used by external tools.

### Use Cases
| Annotation | Used By |
|-----------|---------|
| `kubernetes.io/ingress.class: "nginx"` | Ingress controller |
| `prometheus.io/scrape: "true"` | Prometheus auto-discovery |
| Build/commit SHA, author, timestamp | Your CI/CD pipeline |
| `description: "This pod runs the payment API"` | Documentation |

### Adding Annotations
```yaml
metadata:
  annotations:
    description: "Production payment service"
    git-commit: "abc123f"
    team: "backend"
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
```

## Labels vs Annotations

| | Labels | Annotations |
|---|--------|------------|
| **Purpose** | Identify and select objects | Store metadata and config |
| **Used by selectors** | ✅ Yes | ❌ No |
| **Value size limit** | 63 characters | 256 KB |
| **Used by K8s internally** | ✅ Yes (Services, Deployments) | ❌ Mostly by external tools |
| **Example** | `app: nginx` | `description: "payment service v2"` |

## Key Commands

```bash
# Show labels
kubectl get pods --show-labels
kubectl get pods -L app,tier                  # Show specific labels as columns

# Add/update labels on existing resources
kubectl label pod nginx-pod version=v2
kubectl label pod nginx-pod version=v3 --overwrite

# Remove a label
kubectl label pod nginx-pod version-

# Add an annotation
kubectl annotate pod nginx-pod description="test pod"

# Remove an annotation
kubectl annotate pod nginx-pod description-

# Filter by label
kubectl get pods -l app=nginx
kubectl get all -l tier=frontend
```
