# Adding an Application

Adding an application involves a few steps to ensure that ArgoCD has everything it needs to deploy an application. We
have setup ArgoCD to look at any applications (`yml` files) that are located in the following directory:
- `argocd/apps/applications`

You can either create an application that leverages Helm charts, or just plain old Kubernetes config. Adding a new 
`Application` to that folder will trigger ArgoCD to do a deployment. Below you will find more detailed instructions for
both Helm and plain Kubernetes config.

## Helm

> Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

> Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste.

We can leverage Helm charts to create re-usable sets of Kubernetes configuration. This is great if you want to re-use
config among a few different applications. An example of this can be found in the `gotth-stack` application which uses
the `gotth-app` Helm chart. If we wanted to create more applications that leverage that same Golang stack, we simple use
the same Helm chart which makes development simple.

### 1. Create Application YML

Below you will find a template for an application that uses a Helm chart as part of the config. This should be put in
the `argocd/apps/applications` directory.

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <application_name>
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/dobsondev/homelab.git
    path: argocd/helm/<helm_chart>
    targetRevision: main
    helm:
      valueFiles:
        # Note, this is relative path to the chart above (argocd/helm/<chart>)
        - ../../apps/values/<application_name>.yml
  destination:
    namespace: <application_name>
    name: in-cluster
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

In the above template, you will want to replace the following:
- `<application_name>` : The name of the application
- `<helm_chart>` : The directory of the Helm chart that this application will use

### 2. Create Values YML

This values YML should be named after the `<application_name>` from above and put in the `argocd/apps/values` directory.
The actual content of this file will depend on what Helm chart you are using as the base of your application.

If you are not using Helm to template your application, then you don't need a values file.

### 3. Create Helm Chart

If needed, you can create a new Helm chart for your application to use, simply put it in the `argocd/helm` directory.
This is the perfered method if you plan on re-using your config, for example there is a Helm chart for GoTTH Stack 
Golang applications in this repo so I can create as many as I want by re-using the same Helm chart.

The Helm documentation can be found here if you need more direction on creating a chart:
- https://helm.sh/docs/

## Plain Kubernetes Config

Rather than using Helm, if you have a simple application you can deploy it using ArgoCD with plain Kubernetes config. An
example of this can be seen with the `nginx-example` application.

### 1. Create Application YML

Below you will find a template for an application that can be deployed via ArgoCD. This should be put in the 
`argocd/apps/applications` directory.

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <application_name>
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/dobsondev/homelab.git
    targetRevision: main
    path: argocd/k8s/<application_name>
  destination:
    namespace: <application_name>
    name: in-cluster
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

In the above template, you will want to replace the following:
- `<application_name>` : The name of the application as well as the directory that contains the application

### 2. Create the Kubernetes Config

Add all of your Kubernetes config (`deployment.yml`, `service.yml`, `ingress.yml`, etc...) to the `argocd/k8s/<application_name>`
directory where `<application_name>` is the same as above. ArgoCD will look into this folder and deploy everything for
you.