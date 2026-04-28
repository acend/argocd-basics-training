---
title: "App of Apps"
weight: 61
onlyWhen: app-of-apps
---
The [App of apps](https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/#app-of-apps-pattern) pattern is a declarative specification of one ArgoCD app that consists only of **other ArgoCD applications**.
This way we have the possibility to deploy multiple apps within just one single App definition.

In Lab [2.7]({{< ref  "02" >}}) you learnt how ArgoCD stores its state as Application Custom Resources. The basic idea behind the App of Apps pattern therefore is to store those Application Custom Resources within an Argo CD Application.

First let us examine our ArgoCD example repository with the child applications.

```
.
├── app-of-apps
│   ├── app1.yaml
│   ├── app2.yaml
│   └── app3.yaml
```

As we can see the directory consists of three ArgoCD applications. Each of them has its own source repository pointing to the corresponding repository containing a kubernetes deployment yaml file.


## {{% task %}} Specify the Application Resources

To deploy the app of apps into our namespace we need to edit the three application custom Resources (`app-of-apps/apps/*`):

* Replace all occurrences `<username>` in the **three** yaml files.
* Set the correct `<repourl>` eg. (`https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git`)


<!-- markdownlint-disable -->
{{< highlight YAML "hl_lines=4 8 13" >}}
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <username>-app-of-apps-1
  namespace: argocd
spec:
  destination:
    namespace: <username>
    name: in-cluster
  project: default
  source:
    path: app-of-apps-applications/app1
    repoURL: <repourl>
    targetRevision: HEAD
{{< / highlight >}}
<!-- markdownlint-restore -->

Make sure to also commit and push your changes to the git repository.

{{% alert title="Note" color="info" %}}
Please make sure, to update all three application files
{{% /alert %}}

{{% details title="Hint" %}}
```bash
git add .
git commit -m "Update URLs for App of App"
git push
```
{{% /details %}}


## {{% task %}} Create Argo CD Application

Now let us create the parent Application which deploys our child applications as Custom Resources.
Note the three paramters

* `--sync-policy automated` Set the sync policy to automated. This ensures that our child applications will be created and synced per default
* `--self-heal` Enable self heal and ensure that the parent application reconcile the child application
* `--auto-prune` Ensure that if the parent application gets deleted, it also delete the their child applications

```bash
argocd app create argo-aoa-$USER --repo https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --path 'app-of-apps' --dest-server https://kubernetes.default.svc --dest-namespace $USER --sync-policy automated --self-heal --auto-prune
```

Expected output: `application 'argo-aoa-<username>' created`

Explore the Argo parent application in the web UI.

As you can see our newly created parent app consits of another three apps.

Note that the child applications resources are not synced automatically. This is because an ArgoCD application only synces their direct child resources. To sync the child apps, either click on sync in the ArgoCD UI or set the sync policy to automed.  


## {{% task %}} Delete the Application

Delete the application after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
```bash
argocd app delete argo-aoa-$USER
```
{{% /details %}}

