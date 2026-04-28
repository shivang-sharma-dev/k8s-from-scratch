# Ingress

## What is Ingress?

An Ingress is an API object that manages **external HTTP/HTTPS access** to services inside the cluster. It provides:

- **Host-based routing** — route `app.example.com` to one service and `api.example.com` to another
- **Path-based routing** — route `/app` to frontend and `/api` to backend
- **TLS termination** — handle HTTPS certificates at the edge
- **Single entry point** — one external IP instead of one per service

## Why Not Just Use LoadBalancer Services?

| Approach | Problem |
|----------|---------|
| **LoadBalancer per service** | Each service gets its own cloud LB → expensive, hard to manage |
| **Ingress** | One LB → Ingress Controller → routes to many services by host/path |

```
Without Ingress:                          With Ingress:
Cloud LB → Service A                     Cloud LB → Ingress Controller
Cloud LB → Service B                                ├── /app  → Service A
Cloud LB → Service C                                ├── /api  → Service B
(3 LBs = $$$)                                       └── /docs → Service C
                                                     (1 LB = $)
```

## Ingress Controller (Required)

**An Ingress resource does NOTHING by itself.** You need an **Ingress Controller** — a pod that watches for Ingress resources and configures routing rules.

Popular Ingress Controllers:

| Controller | Backed By |
|-----------|-----------|
| **NGINX Ingress Controller** | Nginx (most common) |
| **Traefik** | Traefik proxy |
| **HAProxy** | HAProxy |
| **AWS ALB Ingress** | AWS Application Load Balancer |
| **Istio Gateway** | Envoy / Istio service mesh |

### Installing NGINX Ingress Controller on Kind:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

## Routing Examples

### Path-Based Routing

Route different URL paths to different services:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /frontend
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 8080
```

### Host-Based Routing

Route different domains to different services:

```yaml
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 8080
```

## TLS / HTTPS

```yaml
spec:
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls-secret     # Secret containing the certificate
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

Create the TLS secret:
```bash
kubectl create secret tls myapp-tls-secret --cert=tls.crt --key=tls.key
```

## Path Types

| Type | Behavior |
|------|----------|
| `Prefix` | Matches URL path prefix. `/api` matches `/api`, `/api/users`, `/api/v1` |
| `Exact` | Matches the exact path only. `/api` matches only `/api`, NOT `/api/users` |

## Key Commands

```bash
kubectl get ingress -n <ns>
kubectl describe ingress <name>
kubectl get ingressclass                  # See installed ingress controllers
```
