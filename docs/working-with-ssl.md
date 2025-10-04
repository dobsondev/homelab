# Working with SSL on the Cluster

Everything is this README should be done after doing the [SSL setup](./ssl-setup.md) and getting ArgoCD installed and you've started uisng the GitOps approach to deploy things.

A reminder of the technologies we are using for SSL:

- [Traefik](https://traefik.io/solutions/kubernetes-ingress)
- [cert-manager](https://cert-manager.io/)
- [Cloudflare](https://www.cloudflare.com/)
- [Let's Encrypt](https://letsencrypt.org/)

## ArgoCD Ingress with SSL

You can following the instructions here to setup the Ingress for ArgoCD:
- [ArgoCD Ingress](argocd.md#argocd-ingress)

## Other Applications Ingress

When you create an ingress with the following configuration:

- `cert-manager.io/cluster-issuer: letsencrypt-production` annotation
- `secretName: <whatever>-dobson-dev-tls`

`cert-manager` will automatically:

1. Creates a Certificate resource **in that namespace**
2. Requests a certificate for the specific hostname (e.g., `gotth.homelab.dobson.dev`)
3. Stores it in a secret named `<SERVICE_NAME>-tls` **in that namespace** (e.g. `gotth-stack-tls`)

So what actually happens is that multiple secrets with the same name but in different namespaces are created:

- `wildcard-tls` in `default` namespace (manual wildcard cert)
- `gotth-stack-tls` in `gotth-stack` namespace (auto-created for `gotth.homelab.dobson.dev`)
- `traefik-tls` in `kube-system` namespace (auto-created for `traefik.homelab.dobson.dev`)
- `nginx-tls` in `nginx-example` namespace (auto-created for `nginx.homelab.dobson.dev`)

### Helm Behaviour

When you don't specify a namespace in your Helm templates, Helm automatically uses the namespace where you're installing  the release. That's why Helm applications such as the `gotth-stack` application works seamlessly - all resources get deployed into the `gotth-stack` namespace automatically.