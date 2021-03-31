---
title: "7.1 Helm"
weight: 71
sectionnumber: 7.1
---

This lab explains how to use [Helm](https://helm.sh/) as manifest format together with Argo CD.

{{% alert title="Note" color="primary" %}}
This lab is work in progress
{{% /alert %}}


## Helm

[Helm](https://github.com/helm/helm) is a [Cloud Native Foundation](https://www.cncf.io/) project to define, install and manage applications in Kubernetes.

It can be used to package multiple Kubernetes resources into a single logical deployment unit.

Helm Charts are configured using `values.yaml` files. (e.g. images, image tags, hostnames, ...).

When using `helm` charts together with Argo CD we can specify the `values.yaml` like this:

```bash
argocd app set argo-helm-$LAB_USER --values values-production.yaml
```
The `--values` flag can be repeated to support multiple values files.

{{% alert title="Info" color="primary" %}}
Values files must be in the same git repository as the Helm chart. The files can be in a different location in which case it can be accessed using a relative path relative to the root directory of the Helm chart.
{{% /alert %}}


### Helm Parameters

Similar to when using `helm` directly (`helm install <release> --set replicaCount=2 ./mychart --namespace <namespace>`), you are able to overwrite values from the values.yaml, by setting parameters.

```bash
argocd app set argo-helm-$LAB_USER -p replicaCount=2
```

{{% alert title="Warning" color="secondary" %}}
Argo CD provides a mechanism to override the parameters of Argo CD applications. [The Argo CD parameter overrides](https://argoproj.github.io/argo-cd/user-guide/parameters/) feature is provided mainly as a convenience to developers and is intended to be used in dev/test environments, vs. production environments.

Many consider this feature as anti-pattern to GitOps. So only use this feature when no other option is available!
{{% /alert %}}


### Helm Release Name

By default, the Helm release name is equal to the Application name to which it belongs. Sometimes, especially on a centralised ArgoCD, you may want to override that name, and it is possible with the `release-name` flag on the cli:

```bash
argocd app set argo-helm-$LAB_USER --release-name <release>
```

{{% alert title="Warning" color="secondary" %}}
Please note that overriding the Helm release name might cause problems when the chart you are deploying is using the app.kubernetes.io/instance label. ArgoCD injects this label with the value of the Application name for tracking purposes.
{{% /alert %}}


### Helm Hooks

[Helm hooks](https://helm.sh/docs/topics/charts_hooks/) are similar to the Argo CD Hooks from [lab 4](../../04/).


### Further Docs

Read more about the helm integration in the [official documentation](https://argoproj.github.io/argo-cd/user-guide/helm/)


## Task {{% param sectionnumber %}}.1: Deploy the simple-example as Helm Chart

Use the [simple-example](https://github.com/acend/argocd-training-examples/tree/master/example-app) from Lab 1 and implement it and create a new Helm Chart.

You can find additional examples [here](https://github.com/argoproj/argocd-example-apps).
