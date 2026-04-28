# 🧪 Playground

Scratch space for hands-on experiments. Break things here, learn from it.

## Structure

```
playground/
├── sscluster/          # 3-node kind cluster practice YAMLs
│   ├── namespace.yml
│   ├── pod.yml
│   ├── deployment.yml
│   ├── replicasets.yml
│   └── daemonsets.yml
└── <your-next-lab>/    # Create a new folder per experiment
```

## Lab Ideas to Try

- [ ] Create a pod and exec into it
- [ ] Deploy nginx with 3 replicas and trigger a rolling update
- [ ] Scale a deployment up/down and watch pods
- [ ] Create a Service and access nginx via port-forward
- [ ] Apply a ConfigMap and read it as an env variable inside a pod
- [ ] Write a Job that runs a simple script once
- [ ] Write a CronJob that runs every minute
- [ ] Apply a DaemonSet and verify one pod per node
- [ ] Create a NetworkPolicy that blocks all ingress to a pod

## Cluster Setup

```bash
# Start your 3-node kind cluster
kind create cluster --config=../../3nodes.yml --name prac

# Set kubectl context
kubectl cluster-info --context kind-prac

# Apply everything in sscluster/
kubectl apply -f sscluster/
```
