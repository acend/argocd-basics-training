---
title: "6.1 App of Apps"
weight: 601
sectionnumber: 6.1
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


## Task {{% param sectionnumber %}}.1: Specify the Application Resources

To deploy the app of apps into our namespace we need to edit the three application custom Resources (`app-of-apps/apps/*`):

* Replace all occurrences `<username>` in the **three** yaml files.
* Set the correct `<repourl>` eg. (`https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git`)

<!-- markdownlint-disable -->
{{< highlight YAML "hl_lines=4 10 15" >}}
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <username>-app-of-apps-1
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io
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

{{% alert title="Note" color="primary" %}}Please make sure, to update all three application files{{% /alert %}}

{{% details title="Hint" %}}
```bash
git add .
git commit -m "Update URLs for App of App"
git push
```
{{% /details %}}


## Task {{% param sectionnumber %}}.2: Create Argo CD Application

Now let us create the parent Application which deploys our child applications as Custom Resources.

```bash
argocd app create argo-aoa-$STUDENT --repo https://{{% param giteaUrl %}}/$STUDENT/argocd-training-examples.git --path 'app-of-apps' --dest-server https://kubernetes.default.svc --dest-namespace $STUDENT
```

Expected output: `application 'argo-aoa-<username>' created`

Explore the Argo parent application in the web UI.

As you can see our newly created parent app consits of another three apps.


## Task {{% param sectionnumber %}}.3: Delete the Application

Delete the application after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
```bash
argocd app delete argo-aoa-$STUDENT
```
{{% /details %}}

