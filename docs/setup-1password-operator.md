# 1Password Operator

> The 1Password Connect Kubernetes Operator integrates Kubernetes Secrets with 1Password via a Connect server. It allows you to:
> - Create Kubernetes Secrets from 1Password items and load them into Kubernetes deployments.
> - Automatically restart deployments when 1Password items update.

## Prerequisites

Before setting up the operator you need to create a 1Password Connect server and obtain two things:

1. A **credentials JSON file** (`1password-credentials.json`)
2. A **Connect token**

To get these, log in to [1password.com](https://1password.com), navigate to **Developer Tools → Connect a Server**, and follow the setup flow. Download the credentials file and save the token somewhere safe — you will need both below.

## Setup

### 1. Create the ArgoCD Application

Create the following file at `argocd/1password-operator.yml`. This file contains the `credentials_base64` secret so it **must not be committed to Git** — in this repo it is added to the `.gitignore`. DO NOT COMMIT IT WHATEVER YOU DO!

```bash
echo "argocd/1password-operator.yml" >> .gitignore
```

To get the base64-encoded value of your credentials file:

```bash
cat $HOME/Downloads/1password-credentials.json | base64 | tr -d '\n'
```

Then create the file with that value:

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
          credentials_base64: <base64-encoded-credentials>
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

Apply it manually since it is not committed to Git:

```bash
kubectl apply -f argocd/1password-operator.yml
```

### 2. Create the Operator Token Secret

Create the Kubernetes secret containing your Connect token:

```bash
kubectl create secret generic onepassword-operator-token \
  --from-literal=token=<your-connect-token> \
  -n onepassword
```

### 3. Verify the Operator is Running

Check that the Connect server and operator pods are healthy:

```bash
kubectl get pods -n onepassword
```

You should see two running pods — `onepassword-connect` (with 2/2 containers) and `onepassword-connect-operator`.

Verify the Connect server initialized successfully by checking the logs:

```bash
kubectl logs -n onepassword -l app=onepassword-connect -c connect-sync --tail=20
```

You should see health check responses with no errors. If you see `invalid character 'o' looking for beginning of value` the credentials were not loaded correctly — double check the base64 value in your Application manifest.

## Usage

Once the operator is running, you can create Kubernetes Secrets from 1Password items by applying a `OnePasswordItem` CRD. These are safe to commit to Git as they contain no secret values — only a reference path to the item in 1Password.

```yaml
apiVersion: onepassword.com/v1
kind: OnePasswordItem
metadata:
  name: my-secret
  namespace: my-namespace
spec:
  itemPath: "vaults/<vault-name>/items/<item-id>"
```

The operator will automatically create a corresponding Kubernetes Secret with fields matching the field names in your 1Password item. You can find the item path using the 1Password CLI:

```bash
# List vaults
op vault list

# List items in a vault
op item list --vault <vault-name>

# Get item details including field names
op item get <item-id> --vault <vault-name>
```