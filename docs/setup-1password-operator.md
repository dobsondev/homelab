# 1Password

> The 1Password Connect Kubernetes Operator integrates Kubernetes Secrets with 1Password with one or more Connect servers. It allows you to:
> - Create Kubernetes Secrets from 1Password items and load them into Kubernetes deployments.
> - Automatically restart deployments when 1Password items update.

Go to 1password.com, login and then navigate to "Developer", "Connect a Server", and then setup everything. You should get a credentails JSON file and a token from 1Password that you will need to use to setup the operator.

Create the following file:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: onepassword-operator
  namespace: argocd
spec:
  project: default
  source:
    chart: connect
    repoURL: https://1password.github.io/connect-helm-charts
    targetRevision: 2.4.1
    helm:
      values: |
        connect:
          credentials_base64: <your-base64-value>
        operator:
          create: true
          token:
            name: onepassword-operator-token
  destination:
    server: https://kubernetes.default.svc
    namespace: onepassword
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Then apply it manually. You need to do this because it contains the `credentials_base64` which is a secret.

```bash
kubectl create secret generic onepassword-operator-token \
  --from-literal=token=<your-connect-token> \
  -n onepassword
```