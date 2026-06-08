---
title: "Tracking and Deployment Strategies"
weight: 81
onlyWhen: tracking-and-deployment-strategies
---

If you are using ArgoCD with Git or Helm tools, ArgoCD gives you the availability to configure different tracking and deployment strategies.


## Helm

For Helm you can use Semver versions either to pin to a specific version or to track minor / patch changes.

| Use Case                                   | How                    | Examples                |
|--------------------------------------------|------------------------|-------------------------|
| Pin to a version (e.g. in production)      | Use the version number | 1.2.0                   |
| Track patches (e.g. in pre-production)     | Use a range            | 1.2.* or >=1.2.0 <1.3.0 |
| Track minor releases (e.g. in QA)          | Use a range            | 1.* or >=1.0.0 <2.0.0   |
| Use the latest (e.g. in local development) | Use star range         | * or >=0.0.0            |


## Git


| Use Case                                   | How                                                                                     | Notes                      |
|--------------------------------------------|-----------------------------------------------------------------------------------------|----------------------------|
| Pin to a specific release                  | Tag the commit (e.g. v1.2.0) and use that tag, or use a commit SHA.                    | See commit and version pinning. |
| Track a branch                             | Use a branch name or HEAD.                                                              | See HEAD / Branch Tracking |


### Head / Branch Tracking

Either a branch name or a symbolic reference (like HEAD). ArgoCD will track the latest commit on that branch and sync whenever it changes.
This is a common pattern for environments that should always reflect the latest state of a branch — including production setups where the branch itself represents the desired state (e.g. a `release` or `main` branch).


### Commit and Version Pinning

The state at the specified Git tag or commit will be applied to the cluster. Pinning can be achieved in two ways: a semver Git tag (e.g. v1.2.0) or a full commit SHA. Git tags are typically easier to work with and still human-readable, while a commit SHA gives you the strongest immutability guarantee. Version pinning via tags is a reasonable choice when you want explicit, auditable control over what gets deployed.


## {{% task %}} Git version pinning

In this task we're going to configure a version pinning with a Git tag. The goal of this task to show you how to pin a version from a Git tag and therefore freeze the deployment to specific commits.

First we create a Git tag `v1.0.0` and push the tag to the repository. We want to re-create the example application and let it track the created git tag `v1.0.0`.

{{% details title="Hint" %}}

To create and push a Git tag execute the following command:
```bash
git tag v1.0.0
git push origin --tags
```
{{% /details %}}

Re-create the simple application example:
{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app create argo-example-$USER --repo https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $USER
```

To pin the v1.0.0 version tag on our application execute the following command:

```bash
argocd app set argo-example-$USER --revision v1.0.0
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Create a file `tagged-application.yaml` with the following content and apply it:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argo-example-<username>
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
    targetRevision: v1.0.0
    path: example-app
  destination:
    server: https://kubernetes.default.svc
    namespace: <username>
```

```bash
{{% param cliToolName %}} apply -f tagged-application.yaml
```

Finally, sync the application in the web UI.
{{% /onlyWhen %}}

Increase the number of replicas in your file `<workspace>/example-app/deployment.yaml` to 2.
After that commit and push your changes to the Git repository.

{{% details title="Hint" %}}

{{< highlight YAML "hl_lines=2" >}}
...
replicas: 2
...
{{< / highlight >}}

For commiting and pushing your changes to your Git repository, execute follwing command:

```bash
git add example-app
git commit -m "scale deployment replicas to 2"
git push
```

{{% /details %}}

Now you can try to sync your application as follows:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app sync argo-example-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on the `argo-example-<username>` application.
{{% /onlyWhen %}}

Check the number of configured replicas on the app deployment.

{{% details title="Hint" %}}
To see the number of configured replicas execute follwing command:

```bash
kubectl describe deployment simple-example
```

You can see in the command output, the number of replicas didn't changed and remains one.

{{< highlight YAML "hl_lines=1" >}}
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
simple-example   1/1     1            1           2m55s
{{< / highlight >}}


{{% /details %}}

Let ArgoCD pickup the latest change, for that we have to create a new git tag and tell ArgoCD to track it.
Let's create a new Git tag with the patch version `v.1.0.1` and push it to the repsitory. Then update the revision and resync the ArgoCD app.

{{% details title="Hint" %}}
Execute the following command to create and push a new Git tag

```bash
git tag v1.0.1 && git push origin --tags
```

{{% onlyWhenNot no-argocd-cli %}}
Execute the following command to set the revision to our new Git tag `v.1.0.1`.

```bash
argocd app set argo-example-$USER --revision v1.0.1
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Edit `tagged-application.yaml`, change `targetRevision` to `v1.0.1`, then re-apply:

```bash
{{% param cliToolName %}} apply -f tagged-application.yaml
```
{{% /onlyWhen %}}

{{% /details %}}

With the newly created tag, ArgoCD is going to pick up and apply the latest changes and scales up the replica count to 2.
First let us sync the changes and check if the ArgoCD App is in Sync.

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app sync argo-example-$USER
```

Then display the status with following command:

```bash
argocd app get argo-example-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on the `argo-example-<username>` application, then verify the status in the UI.
{{% /onlyWhen %}}

If the app is in sync, you can check the number of replicas of the deployment.


```bash
kubectl describe deployment simple-example
```

Now you can see in the output that the replica count has changed to 2.

```bash
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
simple-example   2/2     2            2           7m43s
```


## {{% task %}} Enable auto-sync and prune

Enable automated sync and pruning before deletion to ensure all managed resources are cleaned up:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app set argo-example-$USER --sync-policy automated
argocd app set argo-example-$USER --self-heal
argocd app set argo-example-$USER --auto-prune
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Edit `tagged-application.yaml` to add automated sync policy, then re-apply:

```yaml
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

```bash
{{% param cliToolName %}} apply -f tagged-application.yaml
```
{{% /onlyWhen %}}


## {{% task %}} Delete the Application


You can cascading delete the ArgoCD Application with the following command:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app delete argo-example-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
```bash
{{% param cliToolName %}} delete application argo-example-$USER -n {{% param argoInfraNamespace %}}
```
{{% /onlyWhen %}}
