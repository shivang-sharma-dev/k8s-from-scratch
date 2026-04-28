# ConfigMaps & Secrets

## The Problem

Hardcoding configuration (database URLs, feature flags, API keys) inside container images is bad:
- You'd need to rebuild the image for every config change
- Secrets in images end up in registry history
- Different environments (dev/staging/prod) need different values

**Solution:** Externalize configuration using **ConfigMaps** and **Secrets**.

---

## ConfigMaps

A ConfigMap stores **non-sensitive** key/value configuration data.

### Creating ConfigMaps

#### Via YAML
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: dev-env
data:
  DATABASE_HOST: "mysql-service"
  DATABASE_PORT: "3306"
  LOG_LEVEL: "info"
  APP_MODE: "development"
  # You can also store entire files:
  nginx.conf: |
    server {
      listen 80;
      server_name localhost;
      location / {
        proxy_pass http://backend:8080;
      }
    }
```

#### Via CLI
```bash
# From literal values
kubectl create configmap app-config --from-literal=LOG_LEVEL=info --from-literal=APP_MODE=dev

# From a file
kubectl create configmap nginx-config --from-file=nginx.conf

# From an env file
kubectl create configmap app-config --from-env-file=.env
```

### Using ConfigMaps in Pods

#### As Environment Variables
```yaml
spec:
  containers:
    - name: app
      env:
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_HOST

      # OR inject ALL keys at once:
      envFrom:
        - configMapRef:
            name: app-config
```

#### As a Volume (File Mount)
```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config        # Each key becomes a file here
  volumes:
    - name: config-vol
      configMap:
        name: app-config
```

Files inside `/etc/config/`:
```
/etc/config/DATABASE_HOST     → contains "mysql-service"
/etc/config/DATABASE_PORT     → contains "3306"
/etc/config/nginx.conf        → contains the full nginx config
```

> **Tip:** Volume-mounted ConfigMaps auto-update when you change the ConfigMap (with a delay). Environment variables do NOT — the pod must restart.

---

## Secrets

A Secret stores **sensitive data** — passwords, API keys, TLS certificates. Values are **base64 encoded** (NOT encrypted by default).

### Creating Secrets

#### Via YAML
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=            # base64 of "admin"
  password: cEBzc3dvcmQxMjM=   # base64 of "p@ssword123"
```

To base64 encode:
```bash
echo -n "admin" | base64           # YWRtaW4=
echo -n "p@ssword123" | base64    # cEBzc3dvcmQxMjM=
```

#### Using stringData (Plain Text — K8s encodes for you)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:                         # Plain text, K8s will base64 encode it
  username: admin
  password: p@ssword123
```

#### Via CLI
```bash
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password='p@ssword123'
```

### Secret Types

| Type | Use Case |
|------|----------|
| `Opaque` | General-purpose (default) |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials |
| `kubernetes.io/tls` | TLS certificate + key |
| `kubernetes.io/basic-auth` | Username/password |

### Using Secrets in Pods

#### As Environment Variables
```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: password
```

#### As a Volume
```yaml
volumes:
  - name: secret-vol
    secret:
      secretName: db-credentials
```

### ⚠️ Secrets Are NOT Encrypted by Default

Base64 is encoding, NOT encryption. Anyone with `kubectl get secret -o yaml` access can decode it.

To actually secure Secrets:
1. **Enable encryption at rest** — encrypt etcd data
2. **Use RBAC** — restrict who can read Secrets
3. **Use external secret managers** — HashiCorp Vault, AWS Secrets Manager, sealed-secrets

---

## Resource Requests & Limits

Control how much CPU and memory a container can use.

```yaml
spec:
  containers:
    - name: app
      resources:
        requests:
          cpu: "250m"         # Guaranteed minimum (250 millicores = 0.25 vCPU)
          memory: "128Mi"     # Guaranteed minimum
        limits:
          cpu: "500m"         # Maximum allowed
          memory: "256Mi"     # Maximum allowed — OOMKilled if exceeded
```

| Field | Meaning |
|-------|---------|
| `requests` | **Minimum guarantee.** Scheduler uses this to find a node with enough capacity. |
| `limits` | **Maximum allowed.** Container is throttled (CPU) or killed (memory) if it exceeds. |

### What Happens When Limits Are Exceeded?

| Resource | Behavior |
|----------|----------|
| **CPU** | Container is throttled (slowed down) — NOT killed |
| **Memory** | Container is **OOMKilled** (Out of Memory Killed) and restarted |

### CPU Units
- `1` = 1 vCPU core
- `500m` = 0.5 core (500 millicores)
- `250m` = 0.25 core

### Memory Units
- `128Mi` = 128 mebibytes
- `1Gi` = 1 gibibyte

## Key Commands

```bash
kubectl get configmaps -n <ns>
kubectl describe configmap <name>
kubectl get secrets -n <ns>
kubectl describe secret <name>
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 -d   # Decode a secret
```
