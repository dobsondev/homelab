# Monitoring Setup (Before ArgoCD GitOps)

For metrics and monitoring the cluster is setup to use:

- **Prometheus** - Time-series database for metrics collection
- **Grafana** - Visualization and dashboards
- `kube-state-metrics` - Kubernetes cluster metrics
- `node-exporter` - Hardware and OS metrics

For logging we have:

- **Loki** - Log aggregation system (pairs well with Grafana)
- **Promtail** - Log collector and shipper to Loki

This is often called the "PLG Stack" (Prometheus, Loki, Grafana) and is the de facto standard for Kubernetes observability in homelabs.

## Create Namespace

```bash
kubectl create namespace monitoring
```

## Create Grafana Admin Password

Create an admin password for Grafana, save it in 1Password and then apply it using the following command:

```bash
kubectl create secret generic grafana-admin-secret \
  --namespace=monitoring \
  --from-literal=admin-user="<USER>" \
  --from-literal=admin-password="$(op read 'op://Private/<id>/password')"
```

At this point you will have everything you need ready to install the Kube Prometheus Stack and Loki Stack via ArgoCD.