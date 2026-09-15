# 1Password Operator

> The 1Password Connect Kubernetes Operator integrates Kubernetes Secrets with 1Password via a Connect server. It allows you to:
> - Create Kubernetes Secrets from 1Password items and load them into Kubernetes deployments.
> - Automatically restart deployments when 1Password items update.

## Prerequisites

You will need to have [Helm](https://helm.sh/docs/intro/install/) installed.

```bash
brew install helm
```

Before setting up the operator you need to create a 1Password Connect server and obtain two things:

1. A **credentials JSON file** (`1password-credentials.json`)
2. A **Connect token**

To get these, log in to [1password.com](https://1password.com), navigate to **Developer Tools → Directory → Kubernetes → Connect**, and follow the setup flow. Save the credentials file and the token somewhere safe (I recommend 1Password so you can just reference them in the install command below) — you will need both below.

## Setup

### 1. Create the ArgoCD Application

Install the 1Password Operator via `Helm` (we will use it in setting up SSL and Monitoring as well). This command uses my 1Password references, so you would have to update for your own:

```bash
helm install connect 1password/connect \
  --namespace onepassword --create-namespace \
  --set-file connect.credentials=<(printf '%s' "$(op read "op://425vsdkvxjp77s7lf4akmols4m/vldwi627lax7mchr2syvo2by4y/1password-credentials.json")") \
  --set operator.create=true \
  --set operator.token.value="$(op read 'op://425vsdkvxjp77s7lf4akmols4m/vldwi627lax7mchr2syvo2by4y/Access Token')"
```

Note: we have to strip the newline character in the `1password-credentials.json` file in order for it to properly import. That's what the `printf` above does.

### 2. Verify the Operator is Running

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

Also check to make sure the sync is complete. That means your credentials file and token were valid:

```bash
kubectl logs -n onepassword deploy/onepassword-connect -c connect-sync | grep "### sync complete ###"
```

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