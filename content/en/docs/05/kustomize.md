---
title: "Kustomize"
weight: 52
onlyWhen: kustomize
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

The parameters can be set as follows:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app set argo-kustomize-$USER --nameprefix=<namePrefix>
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
```yaml
# In the Application spec.source.kustomize section:
namePrefix: <namePrefix>
```
{{% /onlyWhen %}}

{{% alert title="Warning" color="warning" %}}
Again, the [The Argo CD parameter overrides](https://argo-cd.readthedocs.io/en/stable/user-guide/parameters/) feature is provided mainly as a convenience to developers and is intended to be used in dev/test environments, vs. production environments.

Many consider this feature as anti-pattern to GitOps. So only use this feature when no other option is available!
{{% /alert %}}


### Further Docs

Read more about the kustomize integration in the [official documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/)


## {{% task %}} Deploy the simple-example with kustomize

Let's deploy the simple-example from lab 1 using [kustomize](https://github.com/acend/argocd-training-examples/tree/master/kustomize/simple-example).

First you'll have to create a new Argo CD application.

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app create argo-kustomize-$USER --repo https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --path 'kustomize/simple-example' --dest-server https://kubernetes.default.svc --dest-namespace $USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Create a file `argocd-kustomize-application.yaml` with the following content and apply it:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argo-kustomize-<username>
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
    targetRevision: HEAD
    path: kustomize/simple-example
  destination:
    server: https://kubernetes.default.svc
    namespace: <username>
```

```bash
{{% param cliToolName %}} apply -f argocd-kustomize-application.yaml
```
{{% /onlyWhen %}}

Sync the application

{{% details title="Hint" %}}

To sync (deploy) the resources you can simply click sync in the web UI{{% onlyWhen no-argocd-cli %}}.{{% /onlyWhen %}}{{% onlyWhenNot no-argocd-cli %}} or execute the following command:

```bash
argocd app sync argo-kustomize-$USER
```
{{% /onlyWhenNot %}}
{{% /details %}}

And verify the deployment:

```bash
{{% param cliToolName %}} get pod --namespace $USER --watch
```

Tell the application to sync automatically, to enable self-healing and auto-prune

{{% details title="Hint" %}}
{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app set argo-kustomize-$USER --sync-policy automated
argocd app set argo-kustomize-$USER --self-heal
argocd app set argo-kustomize-$USER --auto-prune
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Edit `argocd-kustomize-application.yaml` to add automated sync policy, then re-apply:

```yaml
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

```bash
{{% param cliToolName %}} apply -f argocd-kustomize-application.yaml
```
{{% /onlyWhen %}}
{{% /details %}}


## {{% task %}} Set a configuration parameter

We can set the `kustomize` configuration parameter as follows:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app set argo-kustomize-$USER --nameprefix=acend
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Edit `argocd-kustomize-application.yaml` to add the nameprefix in `spec.source.kustomize`, then re-apply:

```yaml
    kustomize:
      namePrefix: acend
```

```bash
{{% param cliToolName %}} apply -f argocd-kustomize-application.yaml
```
{{% /onlyWhen %}}

And take a look at the application in the web UI{{% onlyWhen no-argocd-cli %}}.{{% /onlyWhen %}}{{% onlyWhenNot no-argocd-cli %}} or using the command line tool

{{% details title="Hint" %}}

```bash
argocd app get argo-kustomize-$USER
```
{{% /details %}}
{{% /onlyWhenNot %}}

{{% alert title="Warning" color="warning" %}}
Only use this way of setting params in dev and test stages. Not for Production!
{{% /alert %}}


## {{% task %}} Create a second application representing the production stage

Let's now also deploy an application for the production stage.

This does mean we deploy an overlay which specifically configures the production stage.

Let's create the production stage Argo CD application (path: `kustomize/overlays-example/overlays/production`) with the name `argo-kustomize-prod-<username>` and enable automated sync, self-healing and pruning.


{{% details title="Hint" %}}

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app create argo-kustomize-prod-$USER --repo https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --path 'kustomize/overlays-example/overlays/production' --dest-server https://kubernetes.default.svc --dest-namespace $USER
argocd app set argo-kustomize-prod-$USER --sync-policy automated
argocd app set argo-kustomize-prod-$USER --self-heal
argocd app set argo-kustomize-prod-$USER --auto-prune
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Create a file `argocd-kustomize-application-prod.yaml` with the following content and apply it:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argo-kustomize-prod-<username>
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
    targetRevision: HEAD
    path: kustomize/overlays-example/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: <username>
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

```bash
{{% param cliToolName %}} apply -f argocd-kustomize-application-prod.yaml
```
{{% /onlyWhen %}}

{{% /details %}}

And verify the deployment:

```bash
{{% param cliToolName %}} get pod --namespace $USER --watch
```


## {{% task %}} Delete the Applications

Delete the applications after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app delete argo-kustomize-$USER
argocd app delete argo-kustomize-prod-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
```bash
{{% param cliToolName %}} delete application argo-kustomize-$USER argo-kustomize-prod-$USER -n {{% param argoInfraNamespace %}}
```
{{% /onlyWhen %}}
{{% /details %}}
