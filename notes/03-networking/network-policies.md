# Network Policies

## What is a NetworkPolicy?

A NetworkPolicy is a **firewall for pods**. By default, all pods in a K8s cluster can talk to every other pod. NetworkPolicies let you control which pods can communicate with which.

## Default Behavior (No NetworkPolicy)

```
Pod A ←→ Pod B ←→ Pod C       All pods can talk to all pods ✅
```

Once you apply a NetworkPolicy to a pod, it becomes **deny by default** for the selected direction (ingress/egress). Only explicitly allowed traffic gets through.

## Deny All Ingress

Block ALL incoming traffic to pods with label `app: database`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-to-db
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress              # We're controlling incoming traffic
  ingress: []              # Empty = allow nothing
```

## Allow Only Specific Pods

Allow traffic to the database ONLY from pods labeled `app: api`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
      ports:
        - port: 5432
          protocol: TCP
```

```
api pod → database:5432    ✅ Allowed
frontend pod → database    ❌ Blocked
```

## Allow from Another Namespace

Allow traffic from any pod in the `monitoring` namespace:

```yaml
ingress:
  - from:
      - namespaceSelector:
          matchLabels:
            name: monitoring
```

## Egress (Outgoing Traffic)

Control what your pods can connect TO:

```yaml
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - port: 5432
    - to:                           # Allow DNS (required for service discovery)
        - namespaceSelector: {}
      ports:
        - port: 53
          protocol: UDP
        - port: 53
          protocol: TCP
```

> **Important:** If you restrict egress, always allow DNS (port 53) or your pods won't be able to resolve service names.

## Common Patterns

### 1. Default Deny Everything (Namespace-wide)
```yaml
spec:
  podSelector: {}           # Empty = applies to ALL pods in namespace
  policyTypes:
    - Ingress
    - Egress
```

### 2. Allow Same-Namespace Only
```yaml
ingress:
  - from:
      - podSelector: {}     # Any pod in the SAME namespace
```

### 3. Allow External Internet Access
```yaml
egress:
  - to:
      - ipBlock:
          cidr: 0.0.0.0/0
          except:
            - 10.0.0.0/8       # Block internal network
```

## Requirements

NetworkPolicies need a **CNI plugin that supports them**:

| CNI Plugin | Supports NetworkPolicy? |
|-----------|------------------------|
| **Calico** | ✅ Yes |
| **Cilium** | ✅ Yes |
| **Weave** | ✅ Yes |
| **Flannel** | ❌ No — does NOT enforce NetworkPolicies |
| **Kindnet** (kind default) | ❌ No |

> If your CNI doesn't support it, NetworkPolicies are silently ignored — no error, just no enforcement.

## Key Commands

```bash
kubectl get networkpolicies -n <ns>
kubectl describe networkpolicy <name>
```
