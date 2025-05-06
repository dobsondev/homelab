# Installing OS

I am using [Ubuntu Server 24.04 LTS](https://ubuntu.com/download/server) on all my machines. It has wide spread community support, regular security updates, good compatability with basically everything, and is lightweight enough for my hardware.

1. Download Ubuntu Server LTS from the Ubuntu site
2. Flash it onto a USB drive using whatever imaging software you like
3. Install the OS on all your computers
  - I like to import SSH keys from GitHub so I don't ever need password auth
  - I like to install Docker (stable) when installing the OS

# Installing K3s

The next three sections go over the installation of K3s and setting up your `kubeconfig` file on another work station. I am assuming you will be doing your work on the cluster from a laptop etc...

## Install K3s Server

K3s Documentation can be found here:
- https://docs.k3s.io/

First, you will want to setup K3s on your main server computer. To do this, run the following command:

```bash
curl -sfL https://get.k3s.io | sh -
```

When this is done, you will find the `kubeconfig` file at:

```
/etc/rancher/k3s/k3s.yaml
```

You can copy that `kubeconfig` file to your work station computer and then just replace the localhost URL with the URL of the server on your network.

For example, if your K3s server had IP `192.168.1.123` on your local network, then:

```yaml
    server: https://127.0.0.1:6443
```

would become:

```yaml
    server: https://192.168.1.123:6443
```

in the `kubeconfig` file you keep on your working machine.

There will also be a `K3S_TOKEN` created when you setup the server which can be found at `/var/lib/rancher/k3s/server/node-token`. Note that down and add it to your password manager, as you will need to use it when setting up the K3s agents.

## Install K3s Agents

Run the following command on your agents to add them as nodes to the cluster:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```

Replace `myserver` with the IP address of your server, and replace `mynodetoken` with the token you copied from the previous step.

## Verify

At this point, you should be able to run `kubectl get nodes` and see your nodes running on the cluster:

```bash
NAME              STATUS   ROLES                  AGE    VERSION
k3s-agent-one     Ready    <none>                 146m   v1.32.3+k3s1
k3s-agent-three   Ready    <none>                 112m   v1.32.3+k3s1
k3s-agent-two     Ready    <none>                 116m   v1.32.3+k3s1
k3s-server        Ready    control-plane,master   167m   v1.32.3+k3s
```