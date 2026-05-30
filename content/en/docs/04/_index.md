---
title: "Sync Phases and Waves"
weight: 4
onlyWhen: sync-phases-and-waves
---

In this Lab you are going to learn about [Sync Phases and Waves](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/).


## Sync Phases and Waves

At a high-level, Argo CD executes the sync operation in the three phases pre-sync, sync and post-sync.

Within each phase you can have one or more waves, that allows you to ensure certain resources are healthy before subsequent resources are synced.

When Argo CD starts a sync, it orders the resources in the following precedence:

* The phase
* The wave they are in (lower values first)
* By kind (e.g. namespaces first)
* By name

It then determines the number of the next wave to apply. This is the first number where any resource is out-of-sync or unhealthy.

It applies resources in that wave.

It repeats this process until all phases and waves are in-sync and healthy.


### How to specify waves and phases

Pre-sync and post-sync can only contain hooks defined on annotations `argocd.argoproj.io/hook: PreSync`.

You can specify the wave in the sync phase by setting an annotation `argocd.argoproj.io/sync-wave`. Hooks and resources are assigned to wave zero by default. The wave can be negative, so you can create a wave that runs before all other resources.


## {{% task %}} Sync Wave Example

Let's now get our hands on a sync wave example.

Create the new application `argo-wave-$USER` with the following command. The Application consist of the following resources, phases and waves:

* PreSync
  * Job: upgrade-sql-schema
* Sync Wave 0
  * Deployment: backend
  * Service: backend
* Sync Wave 1
  * Job: maintenance-page-up
* Sync Wave 2
  * Deployment: frontend
  * Service: frontend
* Sync Wave 3
  * Job: maintenance-page-down

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app create argo-wave-$USER --repo https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --path 'sync-wave' --dest-server https://kubernetes.default.svc --dest-namespace $USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Create a file `argocd-wave-application.yaml` with the following content and apply it:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argo-wave-$USER
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git
    targetRevision: HEAD
    path: sync-wave
  destination:
    server: https://kubernetes.default.svc
    namespace: $USER
```

```bash
{{% param cliToolName %}} apply -f argocd-wave-application.yaml
```
{{% /onlyWhen %}}

Sync the application:

{{% details title="Hint" %}}
{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app sync argo-wave-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on the `argo-wave-$USER` application.
{{% /onlyWhen %}}
{{% /details %}}

And verify the deployment:

```bash
{{% param cliToolName %}} get pod --namespace $USER --watch
```


## {{% task %}} Enable auto-sync and prune

Enable automated sync and pruning before deletion to ensure all managed resources are cleaned up:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app set argo-wave-$USER --sync-policy automated
argocd app set argo-wave-$USER --self-heal
argocd app set argo-wave-$USER --auto-prune
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Edit `argocd-wave-application.yaml` to add automated sync policy, then re-apply:

```yaml
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

```bash
{{% param cliToolName %}} apply -f argocd-wave-application.yaml
```
{{% /onlyWhen %}}


## {{% task %}} Delete the Application

Delete the application after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app delete argo-wave-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
```bash
{{% param cliToolName %}} delete application argo-wave-$USER -n {{% param argoInfraNamespace %}}
```
{{% /onlyWhen %}}
{{% /details %}}
