# Services

## What is a Service?

A Service is a stable way to **expose and access pods**. Since pods are ephemeral (they get new IPs when replaced), you need a Service to provide a **permanent address** that routes traffic to healthy pods.

## The Problem Services Solve

```
Without a Service:
Client → Pod IP (10.244.1.5) → Pod dies → IP gone → Client broken ❌

With a Service:
Client → Service (stable IP/DNS) → Routes to healthy pod → Always works ✅
```

A Service uses **label selectors** to find pods and automatically updates its routing when pods are added/removed.

## Service Types

### 1. ClusterIP (Default)

Accessible **only inside the cluster**. Used for internal pod-to-pod communication.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP              # Default, can be omitted
  selector:
    app: backend               # Routes to pods with this label
  ports:
    - port: 80                 # Port the Service listens on
      targetPort: 8080         # Port the container listens on
```

```
Other Pod → backend-service:80 → Pod (app=backend):8080
```

DNS inside the cluster: `backend-service.<namespace>.svc.cluster.local`

### 2. NodePort

Exposes the service on a **static port on every node's IP**. Accessible from outside the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80                 # Service port (internal)
      targetPort: 80           # Container port
      nodePort: 30080          # External port (range: 30000-32767)
```

```
External → <any-node-ip>:30080 → Service → Pod:80
```

**Best for:** Dev/testing. Not for production (no TLS, no load balancing across nodes).

### 3. LoadBalancer

Provisions a **cloud load balancer** (AWS ALB/NLB, GCP LB, Azure LB) that routes external traffic to your service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  type: LoadBalancer
  selector:
    app: api
  ports:
    - port: 443
      targetPort: 8443
```

```
Internet → Cloud LB (public IP) → Service → Pods
```

**Best for:** Production external-facing services. Only works on cloud providers.

### 4. ExternalName

Maps a service to an **external DNS name**. No proxying, no routing — just a DNS alias.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  type: ExternalName
  externalName: db.example.com
```

Now pods can reach `db.example.com` by calling `database.<namespace>.svc.cluster.local`.

## Port Terminology

```yaml
ports:
  - port: 80            # The port OTHER pods/services use to reach this Service
    targetPort: 8080     # The port YOUR container is actually listening on
    nodePort: 30080      # The port on the NODE (only for NodePort/LoadBalancer)
```

```
External:30080 → Service:80 → Container:8080
```

## Service Discovery (DNS)

Every Service gets an automatic DNS record:

```
<service-name>.<namespace>.svc.cluster.local
```

From within the same namespace, just the service name works:
```bash
curl http://backend-service           # Same namespace
curl http://backend-service.prod      # Different namespace (short form)
curl http://backend-service.prod.svc.cluster.local   # Fully qualified
```

## How Services Route Traffic

Services use **iptables** or **IPVS** rules (managed by `kube-proxy`) to route traffic to pod IPs. They don't actually receive traffic — the kernel redirects it.

A Service also creates an **Endpoints** object that lists the IPs of all matching pods:

```bash
kubectl get endpoints backend-service
# NAME              ENDPOINTS                              
# backend-service   10.244.1.5:8080,10.244.2.3:8080
```

## Key Commands

```bash
kubectl get svc -n <ns>
kubectl describe svc <name>
kubectl get endpoints <svc-name>
kubectl port-forward svc/<name> 8080:80     # Forward local port → service
kubectl expose deployment <name> --port=80 --type=NodePort   # Quick way to create a service
```
