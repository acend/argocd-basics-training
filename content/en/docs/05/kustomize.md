---
title: "5.2 Kustomize"
weight: 52
sectionnumber: 5.2
---

This lab explains how to use [kustomize](https://kustomize.io/)  as manifest format together with Argo CD.


## Kustomize Introduction

[Kustomize](https://kustomize.io/) introduces a template-free way to customize application configuration that simplifies the use of off-the-shelf applications. It is built into `kubectl` and `oc` with the command `kubectl apply -k` or `oc apply -k`.

It uses a concept called overlays, which allows to reduce redundant configuration for multiple stages (e.g. dev, prod, test) without a use of a template language.

Argo CD supports kustomize manifests out of the box.


### Kustomize Overlays

When you want to use Kustomize with an overlay, you have to point the Argo Application to the Overlay


### Kustomize Configuration

The following configuration options are available for Kustomize:

* `namePrefix` is a prefix appended to resources for Kustomize apps
* `nameSuffix` is a suffix appended to resources for Kustomize apps
* `images` is a list of Kustomize image overrides
* `commonLabels` is a string map of an additional labels
* `commonAnnotations` is a string map of an additional annotations

Use the following command to set those parameters:

```bash
argocd app set argo-kustomize-$USER --nameprefix=<namePrefix>
```


### Further Docs

Read more about the kustomize integration in the [official documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/)


## Task {{% param sectionnumber %}}.1: Deploy the simple-example with kustomize

Let's deploy the simple-example from lab 1 using [kustomize](https://github.com/acend/argocd-training-examples/tree/master/kustomize/simple-example).

First you'll have to create a new Argo CD application.

```bash
argocd app create argo-kustomize-$USER --repo https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --path 'kustomize/simple-example' --dest-server https://kubernetes.default.svc --dest-namespace $USER
```

Sync the application

{{% details title="Hint" %}}

To sync (deploy) the resources you can simply click sync in the web UI or execute the following command:

```bash
argocd app sync argo-kustomize-$USER
```
{{% /details %}}

And verify the deployment:

```bash
{{% param cliToolName %}} get pod --namespace $USER --watch
```

Tell the application to sync automatically, to enable self-healing and auto-prune

{{% details title="Hint" %}}
```bash
argocd app set argo-kustomize-$USER --sync-policy automated
argocd app set argo-kustomize-$USER --self-heal
argocd app set argo-kustomize-$USER --auto-prune
```
{{% /details %}}


## Task {{% param sectionnumber %}}.2: Set a configuration parameter

We can set the `kustomize` configuration parameter with the following command:

```bash
argocd app set argo-kustomize-$USER --nameprefix=acend
```

And take a look at the application in the web UI or using the command line tool

{{% details title="Hint" %}}

```bash
argocd app get argo-kustomize-$USER
```
{{% /details %}}

{{% alert title="Warning" color="warning" %}}
Only use this way of setting params in dev and test stages. Not for Production!
{{% /alert %}}


## Task {{% param sectionnumber %}}.3: Create a second application representing the production stage

Let's now also deploy an application for the production stage.

This does mean we deploy an overlay which specifically configures the production stage.

Let's create the production stage Argo CD application (path: `kustomize/overlays-example/overlays/production`) with the name `argo-kustomize-prod-$USER` and enable automated sync, self-healing and pruning.


{{% details title="Hint" %}}

```bash
argocd app create argo-kustomize-prod-$USER --repo https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --path 'kustomize/overlays-example/overlays/production' --dest-server https://kubernetes.default.svc --dest-namespace $USER
argocd app set argo-kustomize-prod-$USER --sync-policy automated
argocd app set argo-kustomize-prod-$USER --self-heal
argocd app set argo-kustomize-prod-$USER --auto-prune
```

{{% /details %}}

And verify the deployment:

```bash
{{% param cliToolName %}} get pod --namespace $USER --watch
```


## Task {{% param sectionnumber %}}.4: Delete the Applications

Delete the applications after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
```bash
argocd app delete argo-kustomize-$USER
argocd app delete argo-kustomize-prod-$USER
```
{{% /details %}}
