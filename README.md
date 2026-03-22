# claude_platform_logging: Centralized Logging Stack on Kubernetes

> P2 Platform — Loki + Grafana + Promtail  
> Part of the kiranch97 Solutions Architecture

---

## Overview

Deploys a **centralized logging stack** on Kubernetes:

- **Promtail** — DaemonSet that tails all pod logs from every node and ships to Loki
- **Loki** — Log aggregation engine (Prometheus but for logs), 10Gi PVC
- **Grafana** — Visualization UI to query logs with LogQL, NodePort 30030

Works alongside `grafana-monitoring` (metrics) to give full observability: **metrics + logs** in one Grafana.

---

## Architecture

```
  Every Pod in Cluster
       | (stdout/stderr)
       v
  +----------------+
  | Promtail       |  DaemonSet on every node
  | (log shipper)  |  tails /var/log/pods/**
  +-------+--------+
          |
          v
  +----------------+
  | Loki           |  Log aggregation + storage
  | port: 3100     |  PVC: 10Gi
  +-------+--------+
          |
          v
  +----------------+
  | Grafana        |  Dashboards + LogQL
  | port: 3000     |  NodePort: 30030
  | PVC: 2Gi       |  Ingress: grafana.local
  +----------------+
```

---

## Directory Structure

```
claude_platform_logging/
├── README.md
└── k8s/
    ├── namespace.yaml
    ├── loki.yaml            # Loki + ConfigMap + PVC + Service
    ├── promtail.yaml        # DaemonSet + RBAC + ConfigMap
    ├── grafana.yaml         # Grafana + PVC + NodePort + Datasource
    └── ingress.yaml         # grafana.local via nginx-ingress
```

---

## Deploy

```bash
git clone https://github.com/kiranch97/claude_platform_logging.git
cd claude_platform_logging
kubectl apply -f k8s/
```

Add to /etc/hosts: `127.0.0.1  grafana.local`

---

## Access

| Service | URL | Credentials |
|---------|-----|-------------|
| Grafana UI | http://localhost:30030 | admin / admin |
| Grafana Ingress | http://grafana.local:30080 | admin / admin |

Loki is auto-configured as a datasource. Go to Explore → select Loki → start querying!

---

## LogQL Quick Reference

```logql
# All logs from mlflow namespace
{namespace="mlflow"}

# Filter for errors
{app="mlflow"} |= "error"

# Logs from all claude projects
{namespace=~"jupyter-ml|mlflow|webapp|logging"}

# Count errors per minute
count_over_time({namespace="mlflow"} |= "error" [1m])
```

---

## Cleanup

```bash
kubectl delete namespace logging
```

## Author
**kiranch97** — Built collaboratively with Claude AI | March 2026
