# Basic Service Helm Chart

A general-purpose Helm chart for deploying containerized services into the homelab Kubernetes cluster. It covers the most common needs — replica management, resource limits, ingress with TLS, horizontal autoscaling, config maps, and environment variables — without requiring you to write raw Kubernetes manifests for every new service.

## Features

- **Deployment** with configurable replica count and image settings
- **Service** supporting `ClusterIP` or `NodePort`
- **Ingress** with automatic TLS via cert-manager (optional)
- **Horizontal Pod Autoscaler** scaling on CPU and/or memory (optional)
- **ConfigMap** for key-value configuration data (optional)
- **Environment variables** injected directly into the container
- **Resource requests and limits** for every deployment

## Values Reference

| Key | Default | Description |
|-----|---------|-------------|
| `name` | `gotth-app` | Name used to prefix all Kubernetes resources |
| `component` | `application` | Value for the `app.kubernetes.io/component` label |
| `replicaCount` | `3` | Number of pod replicas |
| `image.repository` | `ghcr.io/dobsondev/repo-name` | Container image repository |
| `image.tag` | `v0.0.1` | Container image tag |
| `image.pullPolicy` | `IfNotPresent` | Image pull policy |
| `service.type` | `ClusterIP` | Service type (`ClusterIP` or `NodePort`) |
| `service.port` | `3000` | Port exposed by the service and container |
| `service.nodePort` | _(unset)_ | NodePort value (30000–32767), only used when `service.type: NodePort` |
| `ingress.enabled` | `false` | Enable the Ingress resource |
| `ingress.host` | `example.homelab.dobson.dev` | Hostname for the Ingress rule and TLS certificate |
| `resources.limits.cpu` | `500m` | CPU limit |
| `resources.limits.memory` | `512Mi` | Memory limit |
| `resources.requests.cpu` | `100m` | CPU request |
| `resources.requests.memory` | `128Mi` | Memory request |
| `autoscaling.enabled` | `false` | Enable the HorizontalPodAutoscaler |
| `autoscaling.minReplicas` | `3` | Minimum replica count when autoscaling |
| `autoscaling.maxReplicas` | `6` | Maximum replica count when autoscaling |
| `autoscaling.targetCPUUtilizationPercentage` | `80` | CPU utilization target (omit to disable CPU scaling) |
| `autoscaling.targetMemoryUtilizationPercentage` | `80` | Memory utilization target (omit to disable memory scaling) |
| `configMap` | `{}` | Key-value pairs to store in a ConfigMap |
| `env` | `[]` | List of environment variables injected into the container |

## Examples

### Minimal Deployment

The smallest viable values file — just set your image and a name:

```yaml
name: my-service
component: application

replicaCount: 1

image:
  repository: ghcr.io/dobsondev/my-service
  tag: v1.2.3
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8080

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 50m
    memory: 64Mi
```

### Enabling Ingress with TLS

Set `ingress.enabled: true` and provide a hostname. cert-manager will automatically provision a Let's Encrypt certificate using the `letsencrypt-production` cluster issuer.

```yaml
ingress:
  enabled: true
  host: my-service.homelab.dobson.dev
```

### NodePort Service

Use `NodePort` when you need to expose the service directly on a node IP without an ingress controller:

```yaml
service:
  type: NodePort
  port: 3000
  nodePort: 30080
```

### Horizontal Pod Autoscaling

Enable the HPA to automatically scale replicas based on CPU and memory usage. The HPA respects `minReplicas` and `maxReplicas` as its bounds:

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 8
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 75
```

To scale on CPU only, omit `targetMemoryUtilizationPercentage` (and vice versa).

### Environment Variables

Inject environment variables directly into the container using the standard Kubernetes `env` list format:

```yaml
env:
  - name: APP_ENV
    value: production
  - name: LOG_LEVEL
    value: info
  - name: PORT
    value: "3000"
```

### ConfigMap

Store configuration key-value pairs in a ConfigMap. Set `configMap.enabled: true` and add your keys beneath it:

```yaml
configMap:
  enabled: true
  DATABASE_HOST: postgres.default.svc.cluster.local
  CACHE_TTL: "300"
  FEATURE_FLAGS: "new-ui,beta-api"
```

> **Note:** The ConfigMap is created as a standalone resource. To consume it inside your container, reference it from the `env` values using `valueFrom.configMapKeyRef`, or mount it as a volume via a custom overlay.

### Full Example

A production-style values file combining multiple features:

```yaml
name: my-service
component: application

replicaCount: 3

image:
  repository: ghcr.io/dobsondev/my-service
  tag: v2.1.0
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 3000

ingress:
  enabled: true
  host: my-service.homelab.dobson.dev

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 6
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

configMap:
  enabled: true
  APP_ENV: production
  LOG_LEVEL: info

env:
  - name: APP_ENV
    value: production
  - name: SECRET_KEY
    valueFrom:
      secretKeyRef:
        name: my-service-secrets
        key: secret-key
```

## Resource Labeling

All resources created by this chart follow the standard `app.kubernetes.io` label conventions:

| Label | Value |
|-------|-------|
| `app.kubernetes.io/name` | `<name>-<resource-type>` |
| `app.kubernetes.io/instance` | `<name>-<resource-type>-<image-tag>` |
| `app.kubernetes.io/version` | `<image.tag>` |
| `app.kubernetes.io/component` | `<component>` |
| `app.kubernetes.io/part-of` | `<name>` |
| `app.kubernetes.io/managed-by` | `Helm` |
