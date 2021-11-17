---
title: "6. App of Apps"
weight: 6
sectionnumber: 6
---
The App Of Apps pattern is a declarative specification of one ArgoCD app that consists only of other apps.
This way we have the possibility to deploy multiple apps within just one single App definition.

First let us examine our ArgoCD example repository with the child applications.

```
.
├── app-of-apps
│   ├── app1
│   │   └── deployment.yaml
│   ├── app2
│   │   └── deployment.yaml
│   ├── app3
│   │   └── deployment.yaml
│   └── apps
│       ├── app1.yaml
│       ├── app2.yaml
│       └── app3.yaml
```

As we can see the diroctory consists of three ArgoCD applications. Each of them has its own source path pointing on the corresponding directory (app1.yaml -> app1/). Each app directory contains a single kubernetes deployment file.

Here the content of the ArgoCD Application 3

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app-of-apps-3
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    namespace: default
    name: in-cluster
  project: default
  source:
    path: app-of-apps/app3
    repoURL: https://github.com/acend/argocd-training-examples # update repo to your own repo
    targetRevision: HEAD
```

{{% alert title="Note" color="primary" %}}Make sure that the namespace in all of your yamls under `apps` are set to `{{% param argoInfraNamespace %}}`.{{% /alert %}}


Now let us create the parent Application which deploys our child applications.

```bash
argocd app create argo-aoa-$LAB_USER --repo https://{{% param giteaUrl %}}/$LAB_USER/argocd-training-examples.git --path 'app-of-apps/apps' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER
```

Expected output: `application 'argo-aoa-<username>' created`

Explore the Argo parent application in the web UI.

As you can see our newly created parent app consits of another three apps.
![App of apps](appofapps.png)

