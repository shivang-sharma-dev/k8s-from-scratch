# Advanced — CRDs, Operators, KEDA

## Custom Resource Definitions (CRDs)

### What is a CRD?

A CRD lets you **define your own resource types** in Kubernetes. Out of the box, K8s has resources like Pods, Services, and Deployments. CRDs let you add custom ones like `Database`, `Certificate`, `KafkaTopic`, etc.

Once you create a CRD, you can manage your custom resource with `kubectl` just like any built-in resource.

### Example: Define a "Website" resource

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: websites.mycompany.com
spec:
  group: mycompany.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                url:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: websites
    singular: website
    kind: Website
    shortNames: ["ws"]
```

### Use the Custom Resource

```yaml
apiVersion: mycompany.com/v1
kind: Website
metadata:
  name: my-portfolio
spec:
  url: "https://example.com"
  replicas: 3
```

```bash
kubectl get websites
kubectl describe website my-portfolio
```

### Real-World CRDs You'll Encounter

| CRD | Installed By |
|-----|-------------|
| `Certificate` | cert-manager |
| `VirtualService` | Istio |
| `Application` | ArgoCD |
| `ScaledObject` | KEDA |
| `PrometheusRule` | Prometheus Operator |

---

## Operators

### What is an Operator?

An Operator is a **custom controller** that uses CRDs to automate the management of complex applications. It encodes the operational knowledge of running an app (install, configure, upgrade, backup, failover) into software.

### Operator = CRD + Controller

```
CRD defines WHAT you want → "I want a 3-node MySQL cluster"
Controller (Operator) watches and makes it happen → Provisions, configures, monitors, heals
```

### How Operators Work

```
1. You create a Custom Resource   →  kind: MySQLCluster, replicas: 3
2. Operator watches for changes   →  Detects new MySQLCluster resource
3. Operator takes action          →  Creates StatefulSets, Services, ConfigMaps, runs init
4. Operator monitors continuously →  If a node fails, Operator repairs it automatically
```

### Example: Deploying MySQL with an Operator

Instead of manually creating StatefulSets, Services, PVCs, ConfigMaps, and init scripts:

```yaml
apiVersion: mysql.oracle.com/v1
kind: MySQLCluster
metadata:
  name: production-db
spec:
  replicas: 3
  version: "8.0"
  storage:
    size: 100Gi
  backup:
    schedule: "0 2 * * *"
```

The Operator handles everything — provisioning, replication, failover, backups.

### Popular Operators

| Operator | Manages |
|----------|---------|
| **Prometheus Operator** | Prometheus + Grafana + Alertmanager |
| **Strimzi** | Apache Kafka clusters |
| **MySQL Operator** | MySQL InnoDB clusters |
| **PostgreSQL Operator (Zalando)** | PostgreSQL clusters |
| **cert-manager** | TLS certificates |
| **ArgoCD** | GitOps deployments |

### Finding Operators

- **OperatorHub.io** — https://operatorhub.io — catalog of community Operators

---

## KEDA (Kubernetes Event-Driven Autoscaler)

### What is KEDA?

KEDA extends Kubernetes autoscaling beyond CPU/memory. It can scale based on **events** — messages in a queue, database connections, HTTP requests, cron schedules, and more.

### HPA vs KEDA

| Feature | HPA | KEDA |
|---------|-----|------|
| **Metrics** | CPU and memory only | 50+ event sources |
| **Scale to zero** | ❌ Minimum 1 pod | ✅ Can scale to 0 pods |
| **Event sources** | None | Kafka, RabbitMQ, SQS, Redis, Cron, HTTP, etc. |

### How KEDA Works

```
Event Source (e.g., Kafka) → KEDA checks queue depth → Scales Deployment up/down
                                                     → Scale to 0 when queue is empty
```

### Installing KEDA

```bash
helm repo add kedacore https://kedacore.github.io/charts
helm install keda kedacore/keda --namespace keda --create-namespace
```

### Example: Scale Based on RabbitMQ Queue

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: rabbitmq-scaler
spec:
  scaleTargetRef:
    name: worker-deployment
  minReplicaCount: 0                   # Scale to zero when no messages
  maxReplicaCount: 50
  triggers:
    - type: rabbitmq
      metadata:
        queueName: task-queue
        host: amqp://guest:guest@rabbitmq-service:5672
        queueLength: "5"               # 1 pod per 5 messages
```

### Example: Scale on Cron Schedule

```yaml
triggers:
  - type: cron
    metadata:
      timezone: Asia/Kolkata
      start: "0 8 * * *"          # Scale up at 8 AM
      end: "0 20 * * *"           # Scale down at 8 PM
      desiredReplicas: "10"
```

### KEDA Trigger Types (Popular)

| Trigger | What It Monitors |
|---------|-----------------|
| `kafka` | Messages in a Kafka topic |
| `rabbitmq` | Messages in a RabbitMQ queue |
| `aws-sqs-queue` | Messages in an SQS queue |
| `redis` | Redis list/stream length |
| `cron` | Time-based schedule |
| `prometheus` | Prometheus query result |
| `mysql` | MySQL query result |
| `http` | HTTP request rate |
