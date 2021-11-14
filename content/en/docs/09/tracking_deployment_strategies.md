---
title: "9.1 Tracking and Deployment Strategies"
weight: 91
sectionnumber: 9.1
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
| Pin to a version (e.g. in production)      | Either tag the commit with (e.g. v1.2.0) and use that tag, or using commit SHA. | See commit and version pinning.        |
| Use the latest (e.g. in local development) | Use HEAD or master (assuming master is your master branch).                             | See HEAD / Branch Tracking |


### Head / Branch Tracking

Either a branch name or a symbolic reference (like HEAD). For branches ArgoCD will take the latest commit of this branch.
This method is often used in development environment where you want to apply the latest changes.


### Commit and Version Pinning

The state at the specified Git tag or commit will be applied to the cluster. Pinning can achieved in two ways. Eiteher you can specifiy the full semver Git tag (v1.2.0) or a commit SHA. Usually the Git tag offers more flexibility while the commit SHA offers more immutuability. Commit pinning is generally the first choice for production environments.


## Task {{% param sectionnumber %}}.1: Git version pinning

In this task we're going to configure a version pinning with a Git tag. The goal of this task to show you how to pin a version from a Git tag and therefore freeze the deployment to specific commits.

First we create a Git tag `v1.0.0` and push the tag to the repository. We want to re-create the example application and let it track the created git tag `v1.0.0`.

{{% details title="Hint" %}}

To create and push a Git tag execute the following command:
```bash
git tag v1.0.0
git push origin --tags
```


Re-create the simple application example:
```bash
argocd app create argo-example-$LAB_USER --repo https://gitea.labapp.acend.ch/$LAB_USER/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER
```

To pin the v1.0.0 version tag on our application execute the following command:

```bash
argocd app set argo-example-$LAB_USER --revision v1.0.0
```
{{% /details %}}


Increase the number of replicas in your file `<workspace>/example-app/values.yaml` to 2.
After that commit and push your changes to the Git repository.

{{% details title="Hint" %}}

{{< highlight YAML "hl_lines=2" >}}

replicaCount: 2

{{< / highlight >}}

For commiting and pushing your changes to your Git repository, execute follwing command:

```bash
git add . && git commit -m "scale deployment replicas to 2" && git push origin
```

{{% /details %}}

Now you can try to sync your applicaion with following command:

```bash
argocd app sync argo-example-$LAB_USER
```

Check the number of configured replicas on the app deployment.

{{% details title="Hint" %}}
To see the number of configured replicas execute follwing command:

```bash
kubectl describe deployment simple-example
```

You can see in the command output, the number of replicas didn't changed and remains to one.

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

Execute the following command to set the revision to our new Git tag `v.1.0.1`.

```bash
argocd app set argo-example-$LAB_USER --revision v1.0.1
```

{{% /details %}}

With the new created tag, ArgoCD is goingt to pick up and apply the latest changes and scales up the replica count to 2.
First let us sync the changes and check if the ArgoCD App is in Sync.

```bash
argocd app sync argo-example-$LAB_USER
```

Then diplay the status with following command:

```bash
argocd app get argo-example-$LAB_USER
```

If the app is in sync, you can check the number of replicas of the producer deployment.


```bash
kubectl describe deployment producer
```

Now you can see in the output that the replica count has changed to 2.

```bash
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
simple-example   2/2     2            2           7m43s
```


## Task {{% param sectionnumber %}}.2: Delete the Application

You can cascading delete the ArgoCD Application with the following command:

```bash
argocd app delete argo-example-$LAB_USER
```
