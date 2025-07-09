# Adding SSL to the Cluster

We are using the following to setup SSL on the cluster:

- [Traefik](https://traefik.io/solutions/kubernetes-ingress)
- [cert-manager](https://cert-manager.io/)
- [Cloudflare](https://www.cloudflare.com/)
- [Let's Encrypt](https://letsencrypt.org/)

This guide assumes that you have a domain name registered and the DNS is hosted with Cloudflare. This is the easiest
setup which is why I chose to go with this option. Note: the cluster is still not public, we do a DNS challenge to issue
the certificates, so the cluster is entirely local still and not accessible from outside my home network.

## Enable Traefik

Ensure Traefik is enabled (it usually is on K3s):

```bash
kubectl get pods -n kube-system | grep traefik
```

## Enable Traefik Dashboard

We might want to enable the Traefik dashboard just to see what is going on with our config. First, patch the Traefik
config to enable the `--api.insecure.true` flag for Traefik:

```bash
kubectl patch deployment traefik -n kube-system --type='json' -p='[
  {
    "op": "add", 
    "path": "/spec/template/spec/containers/0/args/-",
    "value": "--api.insecure=true"
  }
]'
```

Now, check your Traefik config with the following:

```bash
kubectl get deployment -n kube-system traefik -o jsonpath='{.spec.template.spec.containers[0].args}' | tr ',' '\n'
```

You will want to see the following two entries in the list (note: `--api.dashboard=true` is usually enabled by default
on K3s so we don't usually need to patch it in):

```json
"--api.dashboard=true"
...
"--api.insecure=true"
```

### Add Traefik Dashboard to Load Balancer

Since K3s comes with a built in Load Balancer, we can add the Traefik dashboard to the load balancer by using the 
`traefik-dashboard-lb.yml` file in this repo:

```bash
kubectl apply -f ssl/traefik-dashboard-lb.yml
```

## Install `cert-manager`

Install `cert-manager` with Let's Encrypt:

```bash
# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.2/cert-manager.yaml

# Wait for cert-manager to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/instance=cert-manager -n cert-manager --timeout=300s
```

To make sure you get the most up-to-date version of `cert-manager`, check the installation command on this page:
- https://cert-manager.io/docs/installation/

You can confirm that `cert-manager` was installed using the following command:

```bash
kubectl get nodes -n cert-manager -o wide
```

Additionally, you can ensure the custom resources have been installed by checking the following command:

```bash
kubectl get certificate
```

You should get a message like `No resources found in default namespace.` and NOT something like `error: the server 
doesn't have a resource type "certificate"`.

## Create Staging Issuer

Next, we want to create our staging issuer to make sure everything is working as expected. The staging endpoint of Let's
Encrypt has far more generous rate limiting than the production API endpoint, so we want to make sure things are working
on staging before we move on to prod.

The first thing we need to do is create a Secret with our Cloudflare API token. The API token needs to have the
following permissions:

- `Zone.Zone : Read`
- `Zone.DNS: Edit`

Paste that secret into the the `secret-cloudflare-token.yml` file and apply it using:

```bash
kubectl apply -f ssl/issuers/secret-cloudflare-token.yml
```

**BE SURE TO NOT COMMIT THE SECRET TO VERSION CONTROL!!!**

Once the secret has been created, we can create our issuer:

```bash
kubectl apply -f ssl/issuers/letsencrypt-issuer.yml
```

## Create Staging Certificate

Once the issuer has been created, we can finally create our certificate:

```bash
kubectl apply -f ssl/certs/staging-cert.yml
```

You can follow the progress of the certificate being issued by using:

```bash
kubectl logs -n cert-manager -f <cert-manager-pod>
```

You should receieve a message in the logs that contains `"certificate issued"` when it succeeds. This can take a few
minutes. Once the certificate is ready, you should see it when you run the following command:

```bash
kubectl get certificates
```

It should display with a status of `READY`:

```bash
NAME                         READY   SECRET                           AGE
homelab-example-com-staging  True    homelab-example-com-staging-tls  9m8s
```

## Add Staging Certificate to Traefik Dashboard

Next we want to verify everything worked by adding the certificate to our Traefik dashboard. This can be done by
applying the following config:

```bash
kubectl apply -f ssl/traefik-dashboard-ingress-staging.yml
```

This file is setup to create an ingress using our TLS ceritificate and point it to our load balancer that we have
already created. Once this is done, navigate to your chosen domain name. You will still get a Certificate Authority
error, but you can see the certificate was issued by the staging client which means everything is working.

## Production Certificate

Before getting everything ready for production, let's clean up the staging config:

```bash
kubectl delete -f ssl/traefik-dashboard-ingress-staging.yml
kubectl delete -f ssl/certs/staging-cert.yml
kubectl delete -f ssl/issuers/letsencrypt-staging.yml
```

We now want to do the same steps for our production certificate which is the one we will actually use:

```bash
kubectl apply -f ssl/issuers/letsencrypt-production.yml
kubectl apply -f ssl/certs/production-cert.yml
kubectl apply -f ssl/traefik-dashboard-ingress-production.yml
```

At this point you should be able to visit the domain you setup for the Traefik ingress and it should have a valid
certificate attached.

## ArgoCD Ingress with SSL

At this point, you can following the instructions here to setup the Ingress for ArgoCD:
- [ArgoCD Ingress](argocd.md#argocd-ingress)

## Other Applications Ingress

When you create an ingress with the following configuration:
- `cert-manager.io/cluster-issuer: letsencrypt-production` annotation
- `secretName: homelab-dobson-dev-production-tls`

`cert-manager` will automatically:

1. Creates a Certificate resource **in that namespace**
2. Requests a certificate for the specific hostname (e.g., `gotth.homelab.dobson.dev`)
3. Stores it in a secret named `homelab-dobson-dev-production-tls` **in that namespace**

So what actually happens is that multiple secrets with the same name but in different namespaces are created:

- `homelab-dobson-dev-production-tls` in `default` namespace (manual wildcard cert)
- `homelab-dobson-dev-production-tls` in `gotth-stack` namespace (auto-created for `gotth.homelab.dobson.dev`)
- `homelab-dobson-dev-production-tls` in `kube-system` namespace (auto-created for `traefik.homelab.dobson.dev`)
- `homelab-dobson-dev-production-tls` in `nginx-example` namespace (auto-created for `nginx.homelab.dobson.dev`)

### Helm Behaviour

When you don't specify a namespace in your Helm templates, Helm automatically uses the namespace where you're installing 
the release. That's why Helm applications such as the `gotth-stack` application works seamlessly - all resources get 
deployed into the `gotth-stack` namespace automatically.