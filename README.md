# 🏠 Homelab

This repo contains all of the installation instructions and documentation for my homelab.

## Cluster Information

I am using [Ubuntu Server 24.04 LTS](https://ubuntu.com/download/server) on all my machines. It has wide spread community support, regular security updates, good compatability with basically everything, and is lightweight enough for my hardware.

The cluster is running [K3s](https://k3s.io/). K3s is super easy to install, lightweight and provides me everything I need in terms of a cluster.

## Hardware

I am using a bunch of old 1-litre computers to run the cluster. They are small, low energy, and best of all I got them for free. You can also get them fairly cheaply as refurbished on any site like Amazon.

- HP ProDesk 600 G3 Mini - i5-7500T / 8GB RAM / 240 GB SSD
- HP ProDesk 600 G3 Mini - i5-7500T / 8GB RAM / 240 GB SSD
- HP ProDesk 600 G3 Mini - i5-7500T / 8GB RAM / 240 GB SSD
- Lenovo ThinkCentre M720q - i5-8400T / 8 GB RAM / 240 GB NVMe

I run the three HP computers as K3s agents, and the Lenovo as a K3s server.

## Installed Applications

- [GoTTH Stack Example](https://github.com/dobsondev/GoTTH-stack)
- [NGINX Example](https://nginx.org/)
- [Home Assitant](https://www.home-assistant.io/)

## Installed Tools

- [Argo CD](https://argoproj.github.io/cd/)
- [Traefik](https://traefik.io/solutions/kubernetes-ingress)
- [cert-manager](https://cert-manager.io/)
- [Cloudflare](https://www.cloudflare.com/)
- [Let's Encrypt](https://letsencrypt.org/)
- [Longhorn](https://longhorn.io/)
- [Grafana](https://grafana.com/)
- [Prometheus](https://prometheus.io/)
- [Loki](https://grafana.com/oss/loki/)

## Setup

Follow along with the documentation in this order if you want to roll out your own cluster. The first section is the setup before actually installing ArgoCD and using the GitOps approach to deploy things. This is almost like a preflight set of steps to get everything ready for what we are going to install:

1. [Setup Machines](docs/setup-nodes.md)
2. [Setup SSL](docs/setup-ssl.md)
3. [Setup Monitoring](docs/setup-monitoring.md)
4. [Setup Longhorn](docs/setup-longhorn.md)

At this point, you will install ArgoCD and start using the GitOps approach to deploy things:

- [Setup Argo CD](docs/setup-argocd.md)

## Working with the Cluster

Documentation on how to work with the different parts of the cluster can be found below:

- [Working with Applications](docs/working-with-applications.md)
- [Working with SSL](docs/working-with-ssl.md)
- [Working with Longhorn](docs/working-with-longhorn.md)
- [WIP: Working with Monitoring](docs/working-with-monitoring.md)