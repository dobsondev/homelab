# 1Password

> The 1Password Connect Kubernetes Operator integrates Kubernetes Secrets with 1Password with one or more Connect servers. It allows you to:
> - Create Kubernetes Secrets from 1Password items and load them into Kubernetes deployments.
> - Automatically restart deployments when 1Password items update.

Go to 1password.com, login and then navigate to "Developer", "Connect a Server", and then setup everything. You should get a credentails JSON file and a token from 1Password that you will need to use to setup the operator.

```bash
kubectl create secret generic onepassword-helm-credentials \
  --from-literal=values.yaml="connect:
  credentials_base64: $(cat $HOME/Downloads/1password-credentials.json | base64 | tr -d '\n')" \
  -n argocd
```

```bash
kubectl create secret generic onepassword-operator-token \
  --from-literal=token=<your-connect-token> \
  -n onepassword
```