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

## Installed Applications & Tools

- [Argo CD](https://argoproj.github.io/cd/)
- [Traefik](https://traefik.io/solutions/kubernetes-ingress)
- [cert-manager](https://cert-manager.io/)
- [Cloudflare](https://www.cloudflare.com/)
- [Let's Encrypt](https://letsencrypt.org/)

## Setup

Follow the documentation in this order:

1. [Setup Machines](docs/setup.md)
2. [Setup Argo CD](docs/argocd.md)
3. [Setup SSL](docs/ssl.md)