---
title: "5. Sync Phases and Waves"
weight: 5
sectionnumber: 5
---

In this Lab you are going to learn about [Sync Phases and Waves](https://argoproj.github.io/argo-cd/user-guide/sync-waves/).

{{< youtube zIHe3EVp528 >}}


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


## Task {{% param sectionnumber %}}.1: Sync Wave Example

Let's now get our hands on a sync wave example.

Create the new application `argo-wave-$LAB_USER` with the following command. The Application consist of the following resources, phases and waves:

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


```bash
argocd app create argo-wave-$LAB_USER --repo https://{{% param giteaUrl %}}/$LAB_USER/argocd-training-examples.git --path 'sync-wave' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER
```

Sync the application:

{{% details title="Hint" %}}
```bash
argocd app sync argo-wave-$LAB_USER
```
{{% /details %}}

And verify the deployment:

```bash
{{% param cliToolName %}} get pod --namespace $LAB_USER --watch
```


## Task {{% param sectionnumber %}}.2: Delete the Application

Delete the application after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
```bash
argocd app delete argo-wave-$LAB_USER
```
{{% /details %}}
