# n8n

> n8n (pronounced n-eight-n) is a powerful, fair-code licensed, node-based workflow automation tool that allows users to connect apps, databases, and AI agents to automate tasks without writing extensive code. Short for "nodemation" (node + automation), it enables visual building of complex integrations.

Documentation:
- https://docs.n8n.io/

## Helm Chart

We are leveraging this Helm chart for installing n8n via ArgoCD:
- https://github.com/n8n-io/n8n-hosting/tree/main/charts/n8n

Specifically we are using the "standalone" version which leverages SQLite and has no external dependencies:
- https://github.com/n8n-io/n8n-hosting/blob/main/charts/n8n/examples/standalone.yaml

## Setup

The only setup that is required is to create some secrets for n8n to use:

```bash
kubectl create secret generic n8n-secrets \
  --from-literal=N8N_ENCRYPTION_KEY=$(openssl rand -hex 32) \
  --from-literal=N8N_HOST=localhost \
  --from-literal=N8N_PORT=5678 \
  --from-literal=N8N_PROTOCOL=http \
  -n n8n
```