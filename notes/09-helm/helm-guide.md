# Helm — The Kubernetes Package Manager

## What is Helm?

Helm is a **package manager for Kubernetes** — think of it like `apt` for Ubuntu or `npm` for Node.js, but for K8s manifests.

Instead of managing 10+ YAML files by hand, you package them into a **Chart** and deploy/upgrade/rollback with a single command.

## Why Helm?

| Problem | Helm Solution |
|---------|--------------|
| 15 YAML files to deploy one app | Package all files into one Chart |
| Different config for dev/staging/prod | Use `values.yaml` overrides |
| Rollback a broken deployment | `helm rollback` |
| Share K8s configs across teams | Publish Charts to a registry |
| Install complex apps (Prometheus, Grafana) | `helm install prometheus prometheus-community/kube-prometheus-stack` |

## Key Concepts

| Concept | What It Is |
|---------|-----------|
| **Chart** | A package of K8s YAML templates + metadata |
| **Release** | A deployed instance of a Chart in your cluster |
| **Repository** | A server hosting Charts (like DockerHub for images) |
| **Values** | Configuration variables that customize a Chart |

## Chart Structure

```
my-chart/
├── Chart.yaml              # Chart metadata (name, version, description)
├── values.yaml             # Default configuration values
├── templates/              # K8s manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl        # Reusable template snippets
│   └── NOTES.txt           # Post-install instructions
└── charts/                 # Dependencies (sub-charts)
```

### Chart.yaml
```yaml
apiVersion: v2
name: my-app
version: 1.0.0
description: My application Helm chart
type: application
appVersion: "2.1.0"
```

### values.yaml
```yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 250m
    memory: 256Mi
```

### Template Example (templates/deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

`{{ .Values.xxx }}` pulls values from `values.yaml`. You can override them per environment.

## Essential Commands

### Installing
```bash
# Add a repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx

# Install a chart (creates a Release)
helm install my-release bitnami/nginx

# Install with custom values
helm install my-release bitnami/nginx -f custom-values.yaml

# Install with inline overrides
helm install my-release bitnami/nginx --set replicaCount=5
```

### Managing Releases
```bash
helm list                                  # List all releases
helm status my-release                     # Release status
helm upgrade my-release bitnami/nginx      # Upgrade to latest chart version
helm rollback my-release 1                 # Rollback to revision 1
helm uninstall my-release                  # Delete the release
helm history my-release                    # See revision history
```

### Development
```bash
helm create my-chart                       # Scaffold a new chart
helm template my-chart ./my-chart          # Render templates locally (preview)
helm lint ./my-chart                       # Check chart for errors
helm package ./my-chart                    # Package chart into .tgz archive
helm install my-release ./my-chart --dry-run --debug   # Simulate install
```

## Overriding Values Per Environment

```bash
# Dev
helm install app ./my-chart -f values-dev.yaml

# Staging
helm install app ./my-chart -f values-staging.yaml

# Production
helm install app ./my-chart -f values-prod.yaml
```

### values-dev.yaml
```yaml
replicaCount: 1
image:
  tag: "latest"
```

### values-prod.yaml
```yaml
replicaCount: 5
image:
  tag: "2.1.0"
resources:
  requests:
    cpu: 500m
    memory: 512Mi
```

## When to Use Helm

✅ **Use Helm when:**
- Deploying complex apps with many manifests
- You need environment-specific configuration
- You want easy upgrades and rollbacks
- Installing third-party tools (Prometheus, Grafana, cert-manager, ArgoCD)

❌ **Don't need Helm when:**
- You have 1-2 simple YAML files
- You're just learning K8s basics (use raw YAML first)
