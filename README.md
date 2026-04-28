# ☸️ Kubernetes From Scratch

A hands-on K8s learning hub — from core concepts to production-grade projects.

## 📁 Structure

```
k8s-from-scratch/
│
├── notes/                  # Concept notes & YAML examples by topic
│   ├── 01-core-concepts/   # Pods, Namespaces, Labels
│   ├── 02-workloads/       # Deployments, ReplicaSets, DaemonSets, StatefulSets
│   ├── 03-networking/      # Services, Ingress, NetworkPolicies
│   ├── 04-storage/         # PV, PVC, StorageClasses
│   ├── 05-configuration/   # ConfigMaps, Secrets, Resource Limits
│   ├── 06-scheduling/      # Affinity, Taints & Tolerations
│   ├── 07-security/        # RBAC, ServiceAccounts, PodSecurity
│   ├── 08-observability/   # Probes, HPA, Metrics
│   ├── 09-helm/            # Charts, Templates, Values
│   └── 10-advanced/        # CRDs, Operators, KEDA
│
├── cheatsheets/            # Quick reference — kubectl, YAML patterns, troubleshooting
├── resources/              # Books, courses, tools, certifications
├── playground/             # Scratch labs & experiments (sscluster lives here)
├── practice-projects/      # Guided end-to-end projects
├── resume-projects/        # Production-grade portfolio projects
├── interview-prep/         # Common K8s interview questions with answers
│
├── 3nodes.yml              # kind 3-node cluster config
└── req.sh                  # Tool installer script (kubectl, kind, docker)
```

## 🛠️ Tools

`kubectl` · `kind` · `helm` · `k9s` · `kubectx`

## 🎯 Goal

Build deep, practical Kubernetes knowledge and prep for the **CKA** exam.
