---
title: "Orphaned Resources"
weight: 91
onlyWhen: orphaned-resources
---

This lab demonstrates how to find orphaned top-level resources with Argo CD. Orphaned resources are not managed by Argo CD and could be potentially removed from cluster.


## {{% task %}} Create application and project

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app create argo-$USER --repo https://github.com/acend/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $USER
argocd app sync argo-$USER
```

Create new Argo CD project without restrictions for Git source repository (--src) nor destination cluster/namespace (--dest)
```bash
argocd proj create --src "*" --dest "*,*" apps-$USER
```

Enable visualization and monitoring of Orphaned Resources for the newly created project `apps-<username>`
```bash
argocd proj set apps-$USER --orphaned-resources --orphaned-resources-warn
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Create a file `application-orphaned.yaml` with the following content and apply it:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argo-<username>
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: apps-<username>
  source:
    repoURL: https://github.com/acend/argocd-training-examples.git
    targetRevision: HEAD
    path: example-app
  destination:
    server: https://kubernetes.default.svc
    namespace: <username>
```

Create a file `appproject-orphaned.yaml` with the following content and apply it:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: apps-<username>
  namespace: {{% param argoInfraNamespace %}}
spec:
  sourceRepos:
    - '*'
  destinations:
    - server: '*'
      namespace: '*'
  orphanedResources:
    warn: true
```

```bash
{{% param cliToolName %}} apply -f appproject-orphaned.yaml
{{% param cliToolName %}} apply -f application-orphaned.yaml
```

{{% /onlyWhen %}}

{{% alert title="Note" color="info" %}}
{{% onlyWhen no-argocd-cli %}}
The field `orphanedResources` enables the determinability of orphaned resources in Argo CD. After a refresh you will see them in the user interface on the project when selecting the checkbox _Orphaned Resources_.
With `orphanedResources.warn: 'true'`, for each Argo CD application with orphaned resources in the destination namespace a warning will be shown in the user interface.
{{% /onlyWhen %}}
{{% onlyWhenNot no-argocd-cli %}}
The flag `--orphaned-resources` enables the determinability of orphaned resources in Argo CD. After a refresh you will see them in the user interface on the project when selecting the checkbox _Orphaned Resources_.
With the flag `--orphaned-resources-warn` enabled, for each Argo CD application with orphaned resources in the destination namespace a warning will be shown in the user interface.
{{% /onlyWhenNot %}}
{{% /alert %}}

![Orphaned Resources Checkbox in ArgoCD UI](../orphaned-resource-checkbox.png)

{{% onlyWhenNot no-argocd-cli %}}

Assign the application to newly created project:

```bash
argocd app set argo-$USER --project apps-$USER
```

Ensure that the application is now assigned to the new project `apps-<username>`
```bash
argocd app get argo-$USER
```

Refresh the application
```bash
argocd app get --refresh argo-$USER
```
{{% /onlyWhenNot %}}


## {{% task %}} Create orphaned resource

Now create the orphan service `black-hole` in the same target namespace the Argo CD application has:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: black-hole
spec:
  ports:
  - port: 1234
    targetPort: 1234
EOF
```

{{% alert title="Note" color="info" %}}
This service will be detected as orphaned resource because it is not managed by Argo CD. All resources which are managed by Argo CD are marked with the annotation `argocd.argoproj.io/tracking-id` per default.
{{% /alert %}}


{{% onlyWhenNot no-argocd-cli %}}
Print all resources:
```bash
argocd app resources argo-$USER
```

You see in the output that the manually created service `black-hole` is marked as orphaned:
```
GROUP  KIND        NAMESPACE    NAME            ORPHANED
       Service     <username>   simple-example  No
apps   Deployment  <username>   simple-example  No
       Service     <username>   black-hole      Yes
```

When viewing the details of the application you will see the warning about the orphaned resource
```bash
argocd app get --refresh argo-$USER
```

```
...
CONDITION                MESSAGE                               LAST TRANSITION
OrphanedResourceWarning  Application has 1 orphaned resources  2021-09-02 16:20:36 +0200 CEST
...
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Refresh** on the `argo-<username>` application. Navigate to the application details — you will see the `black-hole` service listed as an orphaned resource with an `OrphanedResourceWarning`.
{{% /onlyWhen %}}


## {{% task %}} Enable auto-sync and prune

Enable automated sync and pruning before deletion to ensure all managed resources are cleaned up:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app set argo-$USER --sync-policy automated
argocd app set argo-$USER --self-heal
argocd app set argo-$USER --auto-prune
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Edit `application-orphaned.yaml` to add automated sync policy, then re-apply:

```yaml
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

```bash
{{% param cliToolName %}} apply -f application-orphaned.yaml
```
{{% /onlyWhen %}}


## {{% task %}} Housekeeping

Clean up the resources created in this lab

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app delete argo-$USER -y
argocd proj delete apps-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
```bash
{{% param cliToolName %}} delete application argo-$USER -n {{% param argoInfraNamespace %}}
{{% param cliToolName %}} delete appproject apps-$USER -n {{% param argoInfraNamespace %}}
```
{{% /onlyWhen %}}

Find more detailed information about [Orphaned Resources in the docs](https://argo-cd.readthedocs.io/en/stable/user-guide/orphaned-resources/).
