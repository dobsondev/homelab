# Argo CD

> Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

- https://argoproj.github.io/cd/
- https://argo-cd.readthedocs.io/en/stable/

## Install ArgoCD

Installation instructions for ArgoCD can be found here:
- https://argo-cd.readthedocs.io/en/stable/getting_started/

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Install ArgoCD Custom Resource Definitions

```bash
kubectl apply -k https://github.com/argoproj/argo-cd/manifests/crds\?ref\=stable
```

## Bootstrap Application

Bootstrap the application using the following:

```bash
kubectl apply -f argocd/bootstrap.yml
```

## Access ArgoCD UI

Get the credentials for ArgoCD using the following command:

```bash
argocd admin initial-password -n argocd
```

Note down the intial password (the username is `admin`).

If you do not have the `argocd` CLI tool installed, you can install it via Homebrew:

```bash
brew install argocd
```

Port forward to get the UI to show up on your local machine:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Navigate to [localhost:8080](http://localhost:8080) and login with `admin` and the password you got from the CLI. Note,
you should update your password from the initial one provided as soon as possible. Store the password in a password
manager.

You can use this temporarily to check that everything is working as expected before creating an Ingress using the
documentation below.

## ArgoCD Ingress

**Before adding an Ingress for ArgoCD, ensure you have followed the instructions for setting up [SSL](ssl.md) on the
cluster.**

First, we need to patch ArgoCD to work with HTTP (or in `insecure` mode). We do this by enabling `insecure` mode and
adding the URL we plan on using to the configuration:

```bash
kubectl patch configmap argocd-cmd-params-cm -n argocd --type merge -p '{"data":{"server.insecure":"true"}}'
kubectl patch configmap argocd-cm -n argocd --type merge -p '{"data":{"url":"https://argocd.homelab.dobson.dev"}}'
```

Next, we need to restart the `argocd-server` for the above changes to take effect:

```bash
kubectl rollout restart deployment argocd-server -n argocd
```

You can monitor the progress of the `argocd-server` restart with the following commands:

```bash
# Watch the restart progress
kubectl rollout status deployment argocd-server -n argocd

# Check the pod is running
kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server
```

Finally, we can apply our Ingress:

```bash
kubectl apply -f argocd/argocd-ingress.yml
```

You should now be able to visit the URL assigned in the Ingress and it will work as expected with SSL enabled.