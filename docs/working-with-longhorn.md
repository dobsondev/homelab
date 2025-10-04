# Longhorn

Longhorn is installed via ArgoCD, as Longhorn has documentation on how to achieve this which can be found here:

- https://longhorn.io/docs/1.9.0/deploy/install/install-with-argocd/

Since we are using the GitOps approach for our cluster, this means we simply use the `argocd/apps/longhorn.yml` file and push it to GitHub. Just be sure that you have already done the [Longhorn setup](./longhorn-setup.md) before you install Longhorn via ArgoCD.

Once the install is complete, there is an ingress setup for Longhorn right in the application YAML file. This will generate the URL for accessing the front-end of Longhorn.

## Check Default Storage Class

One thing you will want to do is double check your storage class to see which is the default. Ideally, this should be Longhorn moving forward so that it provisions volumes. You can check using the following command:

```bash
kubectl get storageclass
```

The one marked with `(default)` next to it will be the default. If you happen to have two marked as `(default)`, which was the case for me, then you will get output that looks something like this:

```bash
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  73d
longhorn (default)     driver.longhorn.io      Delete          Immediate              true                   42m
longhorn-static        driver.longhorn.io      Delete          Immediate              true                   42m
```

This can be easily patched with the following command:

```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

K3s sets the `local-path` as `(default)` when it creates the cluster. So we simply want to change that to only use Longhorn. If you run `kubectl get storageclass` after patching then you should get this:

```bash
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path           rancher.io/local-path   Delete          WaitForFirstConsumer   false                  73d
longhorn (default)   driver.longhorn.io      Delete          Immediate              true                   45m
longhorn-static      driver.longhorn.io      Delete          Immediate              true                   45m
```