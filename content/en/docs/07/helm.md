---
title: "7.1 Helm"
weight: 71
sectionnumber: 7.1
---

This lab explains how to use [Helm](https://helm.sh/) as manifest format together with Argo CD.


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
argocd app set argo-helm-$LAB_USER --parameter replicaCount=2
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

Let's deploy the simple-example from lab 1 using a [helm chart](https://github.com/acend/argocd-training-examples/tree/master/helm/simple-example).

First you'll have to create a new Argo CD application.

```bash
argocd app create argo-helm-$LAB_USER --repo https://{{% param giteaUrl %}}/$LAB_USER/argocd-training-examples.git --path 'helm/simple-example' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER
```

Sync the application

{{% details title="Hint" %}}

To sync (deploy) the resources you can simply click sync in the web UI or execute the following command:

```bash
argocd app sync argo-helm-$LAB_USER
```
{{% /details %}}

And verify the deployment:

```bash
oc get pod --namespace $LAB_USER --watch
```

Tell the application to sync automatically, to enable self-healing and auto-prune

{{% details title="Hint" %}}
```bash
argocd app set argo-helm-$LAB_USER --sync-policy automated
argocd app set argo-helm-$LAB_USER --self-heal
argocd app set argo-helm-$LAB_USER --auto-prune
```
{{% /details %}}


## Task {{% param sectionnumber %}}.2: Scale the deployment to 2 replicas

We can set the `helm` parameter with the following command:

```bash
argocd app set argo-helm-$LAB_USER --parameter replicaCount=2
```

{{% alert title="Warning" color="secondary" %}}
Only use this way of setting params in dev and test stages. Not for Production!
{{% /alert %}}

Since the `sync-policy` is set to `automated` the second pod will be deployed immediately.


## Task {{% param sectionnumber %}}.3: Ingress

The prober and production ready way of overwriting values is by doing it in git.

Change the `helm/simple-example/values.yaml` file in your git repository

```yaml
...
ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: helm-<namespace>.<appdomain>
      paths:
      - path: /
  tls: []
...
```

{{% alert title="Note" color="primary" %}}
The `<appdomain>` is the default domain under which your applications will be accessible and is provided by your trainer.
{{% /alert %}}

Commit and push the changes to your repository.

{{% details title="Hint" %}}
```bash
git add helm/simple-example/values.yaml
git commit -m "Expose ingress"
git push
```
{{% /details %}}

Open your Browser and verify whether you can access the application.


## Task {{% param sectionnumber %}}.4: Create a second application representing the production stage

Let's now also deploy an application for the production stage.

Create a new values.yaml file for the production stage: `helm/simple-example/values-production.yaml`

Commit and push the changes to your repository.

{{% details title="Hint" %}}
```bash
git add helm/simple-example/values-production.yaml
git commit -m "Add prod stage"
git push
```
{{% /details %}}


Let's create the production stage Argo CD application with the name `argo-helm-prod-$LAB_USER` and enable automated sync, self-healing and pruning.

{{% details title="Hint" %}}

```bash
argocd app create argo-helm-prod-$LAB_USER --repo https://{{% param giteaUrl %}}/$LAB_USER/argocd-training-examples.git --path 'helm/simple-example' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER
argocd app set argo-helm-prod-$LAB_USER --sync-policy automated
argocd app set argo-helm-prod-$LAB_USER --self-heal
argocd app set argo-helm-prod-$LAB_USER --auto-prune
```

{{% /details %}}

And verify the deployment:

```bash
oc get pod --namespace $LAB_USER --watch
```

Tell the Argo CD app to use the `values-production.yaml` values file

{{% details title="Hint" %}}
```bash
argocd app set argo-helm-$LAB_USER --values values-production.yaml
```
{{% /details %}}

Change for example the ingress hostname to something different in the `values-production.yaml` and verify whether you can access the new hostname.


## Task {{% param sectionnumber %}}.4: Delete the Applications

Delete the applications after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
```bash
argocd app delete argo-helm-$LAB_USER
argocd app delete argo-helm-prod-$LAB_USER
```
{{% /details %}}
