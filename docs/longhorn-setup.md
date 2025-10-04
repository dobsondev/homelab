# Longhorn Setup (Before ArgoCD GitOps)

> Longhorn is a lightweight, reliable and easy-to-use distributed block storage system for Kubernetes.

> Longhorn is free, open source software. Originally developed by Rancher Labs, it is now being developed as a incubating project of the Cloud Native Computing Foundation.

## Installing CLI Tool

Do the following from your client machine (note: I have an Apple Silicon chip, if you do not then your command will change):

```bash
curl -L https://github.com/longhorn/cli/releases/download/v1.9.0/longhornctl-darwin-arm64 -o longhornctl
chmod +x longhornctl
sudo mv ./longhornctl /usr/local/bin/longhornctl
```

## Preflight Check

> This command verifies your Kubernetes cluster environment. It performs a series of checks to ensure your cluster meets the requirements for Longhorn to function properly. These checks can help to identify issues that might prevent  Longhorn from functioning properly.

Again, this should be run from your client machine (not a node on the cluster):

```bash
longhornctl --kube-config ~/.kube/config check preflight
```

If you get any errors, then you need to run the `longhornctl install preflight` command:

```bash
longhornctl --kube-config ~/.kube/config install preflight
```

You should get output similar to this:

```
INFO[2025-10-02T21:18:42-06:00] Initializing preflight installer
INFO[2025-10-02T21:18:42-06:00] Cleaning up preflight installer
INFO[2025-10-02T21:18:42-06:00] Running preflight installer
INFO[2025-10-02T21:18:42-06:00] Installing dependencies with package manager
INFO[2025-10-02T21:19:07-06:00] Installed dependencies with package manager
INFO[2025-10-02T21:19:07-06:00] Retrieved preflight installer result:
k3s-agent-one:
  info:
  - Successfully installed package nfs-common
  - Successfully probed module nfs
  - Successfully probed module dm_crypt
  - Successfully started service iscsid
k3s-agent-three:
  info:
  - Successfully installed package nfs-common
  - Successfully probed module nfs
  - Successfully probed module dm_crypt
  - Successfully started service iscsid
k3s-agent-two:
  info:
  - Successfully installed package nfs-common
  - Successfully probed module nfs
  - Successfully probed module dm_crypt
  - Successfully started service iscsid
k3s-server:
  info:
  - Successfully installed package nfs-common
  - Successfully probed module nfs
  - Successfully probed module dm_crypt
  - Successfully started service iscsid
INFO[2025-10-02T21:19:07-06:00] Cleaning up preflight installer
INFO[2025-10-02T21:19:07-06:00] Completed preflight installer. Use 'longhornctl check preflight' to check the result (on some os a reboot and a new install execution is required first)
```

Following that, run the `longhornctl --kube-config ~/.kube/config check preflight` again and you should not get any errors.

## Resolving Warnings

You might get a few warnings still after running the `longhornctl install preflight` command. I got the following two:

```bash
warn:
  - Kube DNS "coredns" is set with fewer than 2 replicas; consider increasing replica count for high availability
  - multipathd.service is running. Please refer to https://longhorn.io/kb/troubleshooting-volume-with-multipath/ for more information.
```

The first warning can easily be resolved with the following command:

```bash
kubectl scale deployment coredns -n kube-system --replicas=2
```

`multipathd.service` was running on all my nodes which might interfere with Longhorn volumes. The first thing I did was check if I was actually using multipath at all. To do this, I SSH'd into my nodes and then ran the following command:

```bash
sudo multipath -l
```

This will check if there are any multipath devicers configured. I got no results back from running the command so I was safe to simply disable the service on each node:

```bash
sudo systemctl stop multipathd
sudo systemctl disable multipathd
sudo systemctl stop multipathd.socket
sudo systemctl disable multipathd.socket
# Check the status to verify they are disabled:
sudo systemctl status multipathd
sudo systemctl status multipathd.socket
```