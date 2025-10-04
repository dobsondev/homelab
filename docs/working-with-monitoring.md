# Monitoring

For metrics and monitoring the cluster is setup to use:

- **Prometheus** - Time-series database for metrics collection
- **Grafana** - Visualization and dashboards
- `kube-state-metrics` - Kubernetes cluster metrics
- `node-exporter` - Hardware and OS metrics

For logging we have:

- **Loki** - Log aggregation system (pairs well with Grafana)
- **Promtail** - Log collector and shipper to Loki

This is often called the "PLG Stack" (Prometheus, Loki, Grafana) and is the de facto standard for Kubernetes observability in homelabs.