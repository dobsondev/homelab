# Monitoring

For metrics and monitoring the cluster is setup to use:

- **Prometheus** - Time-series database for metrics collection
- **Grafana** - Visualization and dashboards
- `kube-state-metrics` - Kubernetes cluster metrics
- `node-exporter` - Hardware and OS metrics

For logging we have:

- **Loki** - Log aggregation system (pairs well with Grafana)
- **Grafana Alloy** - Unified telemetry collector; ships pod logs to Loki and doubles as the cluster's OTLP gateway in front of Tempo

For tracing we have:

- **Tempo** - Distributed tracing backend, queried from Grafana and cross-linked with Loki logs via trace ID

This is often called the "PLG Stack" (Prometheus, Loki, Grafana) and is the de facto standard for Kubernetes observability in homelabs.

The `monitoring` namespace and the `grafana-admin-secret` (sourced from 1Password via the operator) are both created automatically when ArgoCD syncs `argocd/apps/kube-prometheus-stack.yml` and `argocd/manifests/grafana-admin-secret.yml` — no manual pre-ArgoCD steps needed for either.

## Loki

- https://github.com/grafana/loki/tree/main/production/helm/loki
- https://grafana.com/docs/loki/latest/setup/install/helm/install-monolithic/

## Alloy

- https://grafana.com/docs/alloy/latest/

Alloy's config lives inline in `argocd/apps/alloy.yml`. It runs as a DaemonSet with two jobs:

- **Logs**: discovers every pod on its node and tails the matching kubelet log files, forwarding to Loki — this replaced Promtail, which is EOL/maintenance-mode upstream.
- **OTLP trace gateway**: exposes an OTLP receiver (grpc `:4317` / http `:4318`) that enriches incoming spans with pod/namespace/node metadata before forwarding to Tempo.

Any new app that wants to emit traces should point its `OTEL_EXPORTER_OTLP_ENDPOINT` at Alloy's Service (`alloy.monitoring.svc.cluster.local:4318`) rather than at Tempo directly — see `argocd/apps/go-logging-example-app.yml` for a working example. Sending straight to Alloy means the app gets the automatic k8s-attribute enrichment for free instead of having to set `OTEL_RESOURCE_ATTRIBUTES` itself.