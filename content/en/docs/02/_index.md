---
title: "2. Simple Example"
weight: 2
sectionnumber: 2
---

In this lab you will learn how to deploy a simple application using Argo CD.


## Task {{% param sectionnumber %}}.1: Fork the Git repository

As we are proceeding according to the GitOps principle we need some example resource manifests in a Git repository which we can edit.

Users which have a personal Github account can just fork the Repository [argocd-training-examples](https://github.com/acend/argocd-training-examples) to their personal account. To fork the repository click on the top right of the Github on _Fork_.

All other users can use the provided Gitea installation of the personal lab environment. Visit `https://{{% param giteaUrl %}}/` with your browser and register a new account with your personal username and a password that you can remember ;)

{{% alert title="Note" color="primary" %}}All the cli commands in this chapter must be executed in the terminal of the provided Web IDE.{{% /alert %}}

![Register new User in Gitea](gitea-register.png)

Login with the new user and fork the existing Git repository from Github:

1. Select _Create_ on the top right -> _New Migration_ -> Select _GitHub_
1. Migrate / Clone From URL: https://github.com/acend/argocd-training-examples.git
1. Click _Migrate Repository_

The URL of the newly forked Git repository will look like `https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git`

Set the `LAB_USER` environment variable to your personal user:

```bash
export LAB_USER=<username>
echo $LAB_USER
```

Clone the forked repository to your local workspace:

```bash
git clone https://$LAB_USER@{{% param giteaUrl %}}/$LAB_USER/argocd-training-examples.git
```

... or when forked to Github with:

```bash
https://github.com/<github-username>/argocd-training-examples
```

Change the working directory to the cloned git repository: `cd argocd-training-examples/example-app`

When using the Web IDE: Configure the Git Client and verify the output

```bash
git config user.name "$LAB_USER"
git config user.email "foo@bar.org"
git config --local --list
```


## Task {{% param sectionnumber %}}.2: Deploying the resources with Argo CD

Now we want to deploy the resource manifests contained in the cloned repository with Argo CD to demonstrate the basic features of Argo CD.

To deploy the resources using the Argo CD CLI use the following command:

```bash
argocd app create argo-$LAB_USER --repo https://{{% param giteaUrl %}}/$LAB_USER/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER
```


Expected output: `application 'argo-<username>' created`

{{% alert title="Note" color="primary" %}}We don't need to provide Git credentials because the repository is readable for non-authenticated users as well{{% /alert %}}

{{% alert title="Note" color="primary" %}}If you want to deploy it in a different namespace, make sure the namespaces exists before synching the app{{% /alert %}}

Once the application is created, you can view its status:

```bash
argocd app get argo-$LAB_USER
```

```
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          <username>
URL:                https://{{% param argoCdUrl %}}/applications/argo-<username>
Repo:               https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
Target:
Path:               example-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (5a6f365)
Health Status:      Missing

GROUP  KIND        NAMESPACE    NAME           STATUS     HEALTH   HOOK  MESSAGE
       Service     <username>  simple-example  OutOfSync  Missing
apps   Deployment  <username>  simple-example  OutOfSync  Missing
```


The application status is initially in OutOfSync state. To sync (deploy) the resource manifests, run:

```bash
argocd app sync argo-$LAB_USER
```

This command retrieves the manifests from the git repository and performs a `{{% param cliToolName %}} apply` on them. Because all our manifests has been deployed manually before, no new rollout of them will be triggered on OpenShift. But form now on, all resources are managed by Argo CD. Congrats, the first step in direction GitOps! :)

You should see an output similar to the following lines:

```
TIMESTAMP                  GROUP        KIND   NAMESPACE  NAME            STATUS    HEALTH        HOOK  MESSAGE
2021-03-24T14:19:16+01:00            Service  <username>  simple-example  OutOfSync  Missing
2021-03-24T14:19:16+01:00   apps  Deployment  <username>  simple-example  OutOfSync  Missing
2021-03-24T14:19:16+01:00            Service  <username>  simple-example    Synced  Healthy

Name:               argo-<username>
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          <username>
URL:                https://{{% param argoCdUrl %}}/applications/argo-<username>
Repo:               https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
Target:
Path:               example-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (5a6f365)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      5a6f365d8a65d1dab0761fc0d32b90f7eb480354
Phase:              Succeeded
Start:              2021-03-24 14:19:16 +0100 CET
Finished:           2021-03-24 14:19:16 +0100 CET
Duration:           0s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE    NAME           STATUS  HEALTH       HOOK  MESSAGE
       Service     <username>  simple-example  Synced  Healthy            service/simple-example created
apps   Deployment  <username>  simple-example  Synced  Progressing        deployment.apps/simple-example created
```

Check the Argo CD UI to browse the application and their components. The URL of the Argo CD webinterface will be provided by the teacher.

Application overview in unsynced and synced state

![Application overview (unsynced state)](app-overview-unsynced.png)
![Application overview (synced state)](app-overview-synced.png)

Detailed view of a application in unsynced and synced state

![Application Tree (unsynced state)](app-tree-unsynced.png)

![Application Tree (synced state)](app-tree-sycned.png)


## Task {{% param sectionnumber %}}.3: Automated Sync Policy and Diff

When there is a new commit in your Git repository, the Argo CD application becomes OutOfSync. Let's assume we want to scale up our `Deployment` of the example application from 1 to 2 replicas. We will change this in the Deployment manifest.

Increase the number of replicas in your file `<workspace>/example-app/deployment.yaml` to 2.

```
{{< highlight YAML "hl_lines=6" >}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-example
spec:
  replicas: 2
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: simple-example
  template:
    metadata:
      labels:
        app: simple-example
    spec:
      containers:
      - image: quay.io/acend/example-web-go
        name: simple-example
        ports:
        - containerPort: 5000
{{< / highlight >}}
```


<!--- TODO bb: verify initial authentication against git, describe if necessary -->


Commit the changes and push them to your personal remote Git repository. After the Git push command a password input field will appear at the top of the Web IDE. You need to enter your Gitea password there.

```bash
git add deployment.yaml
git commit -m "Increased replicas to 2"
git push
```

After a successful push you should see the following output

```bash
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 367 bytes | 367.00 KiB/s, done.
Total 4 (delta 3), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
   5a6f365..e2d4bbf  master -> master
```

Check the state of the resources by cli:

```bash
argocd app get argo-$LAB_USER --refresh
```

The parameter `--refresh` triggers an update against the Git repository. Out of the box Git will be polled by Argo CD in a predefined interval (defaults to 3 minutes). To use a synchronous workflow you can use webhooks in Git. These will trigger a synchronization in Argo CD on every push to the repository.

You will see that the Deployment is now OutOfSync:

```
Name:               argo-<username>
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          <username>
URL:                https://{{% param argoCdUrl %}}/applications/argo-<username>
Repo:               https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
Target:
Path:               example-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (e2d4bbf)
Health Status:      Healthy

GROUP  KIND        NAMESPACE    NAME            STATUS     HEALTH   HOOK  MESSAGE
       Service     <username>   simple-example  Synced     Healthy        service/simple-example created
apps   Deployment  <username>   simple-example  OutOfSync  Healthy        deployment.apps/simple-example created
```

When an application is OutOfSync then your deployed 'live state' is no longer the same as the 'target state' which is represented by the resource manifests in the Git repository. You can inspect the differences between live and target state by cli:

```bash
argocd app diff argo-$LAB_USER
```

which should give you an output similar to:

```
===== apps/Deployment <username>/simple-example ======
101c102
<   replicas: 1
---
>   replicas: 2
```

Now open the web console of Argo CD and go to your application. The deployment `simple-example` is marked as 'OutOfSync':

![Application Out-of-Sync](app-replicas-diff-overview.png)

With a click on Deployment > Diff you will see the differences:

![Application Differences](app-replicas-diff-detail.png)


Now click `Sync` on the top left and let the magic happens ;) The application will be scaled up to 2 replicas and the resources are in Sync again.

Double-check the status by cli

```bash
argocd app get argo-$LAB_USER
```

```
Name:               argo-<username>
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          <username>
URL:                https://{{% param argoCdUrl %}}/applications/argo-<username>
Repo:               https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
Target:
Path:               example-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (e2d4bbf)
Health Status:      Healthy

GROUP  KIND        NAMESPACE    NAME            STATUS  HEALTH   HOOK  MESSAGE
       Service     <username>   simple-example  Synced  Healthy        service/simple-example unchanged
apps   Deployment  <username>   simple-example  Synced  Healthy        deployment.apps/simple-example configured
```

Argo CD can automatically sync an application when it detects differences between the desired manifests in Git, and the live state in the cluster. A benefit of automatic sync is that CI/CD pipelines no longer need direct access to the Argo CD API server to perform the deployment. Instead, the pipeline makes a commit and push to the Git repository with the changes to the manifests in the tracking Git repo.

To configure automatic sync run (or use the UI):

```bash
argocd app set argo-$LAB_USER --sync-policy automated
```

From now on Argo CD will automatically apply all resources to Kubernetes every time you commit to the Git repository.

Decrease the replicas count to 1 and push the updated manifest to remote. Wait for a few moments and see check that ArgoCD will scale the deployment of the example app down to 1 replica. The default polling interval is 3 minutes. If you don't want to wait you can force a refresh by clicking `Refresh` in the UI or by cli:

```bash
argocd app get argo-$LAB_USER --refresh
```


## Task {{% param sectionnumber %}}.4: Automatic Self-Healing

By default, changes made to the live cluster will not trigger automatic sync. To enable automatic sync when the live cluster's state deviates from the state defined in Git, run:

```bash
argocd app set argo-$LAB_USER --self-heal
```

Watch the deployment `simple-example` in a separate terminal

```bash
{{% param cliToolName %}} get deployment simple-example -w --namespace=$LAB_USER
```

Let's scale our `simple-example` Deployment and observe whats happening:

```bash
{{% param cliToolName %}} scale deployment simple-example --replicas=3 --namespace=$LAB_USER
```

Argo CD will immediately scale back the `simple-example` Deployment to `1` replicas. You will see the desired replicas count in the watched Deployment.

```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
simple-example   1/1     1            1           114m
simple-example   1/3     1            1           114m
simple-example   1/3     1            1           114m
simple-example   1/3     1            1           114m
simple-example   1/3     3            1           114m
simple-example   1/1     3            1           114m
simple-example   1/1     3            1           114m
simple-example   1/1     3            1           114m
simple-example   1/1     1            1           114m
```

This is a great way to enforce a strict GitOps principle. Changes which are manually made on deployed resource manifests are reverted immediately back to the desired state by the ArgoCD controller.


## Task {{% param sectionnumber %}}.5: Pruning

You probably asked yourself how can I delete deployed resources on the container platform? Argo CD can be configured to delete resources that no longer exist in the Git repository.

First delete the file `svc.yaml` from Git repository and push the changes

```bash
git rm svc.yaml
git add --all && git commit -m 'Removes service' && git push

```

Check the status of the application with

```bash
argocd app get argo-$LAB_USER --refresh
```

You will see that even with auto-sync and self-healing enabled the status is still OutOfSync

```
GROUP  KIND        NAMESPACE    NAME            STATUS     HEALTH   HOOK  MESSAGE
apps   Deployment  <username>   simple-example  Synced     Healthy        deployment.apps/simple-exampleconfigured
       Service     <username>   simple-example  OutOfSync  Healthy
```

Now enable the auto pruning explicitly:

```bash
argocd app set argo-$LAB_USER --auto-prune
```

Recheck the status again

```bash
argocd app get argo-$LAB_USER --refresh
```

```
GROUP  KIND        NAMESPACE    NAME            STATUS     HEALTH   HOOK  MESSAGE
       Service     <username>   simple-example  Succeeded  Pruned         pruned
apps   Deployment  <username>   simple-example  Synced     Healthy        deployment.apps/simple-example unchanged
```

The Service was successfully deleted by Argo CD because the manifest was removed from git. See the HEALTH and MESSAGE of the previous console output.


## Task {{% param sectionnumber %}}.6: State of ArgoCD

Argo CD is largely built stateless. The configuration is persisted as native Kubernetes objects. And those are stored in Kubernetes _etcd_. There is no additional storage layer needed to run ArgoCD. The Redis storage under the hood acts just as a throw-away cache and can be evicted anytime without any data loss.

The configuration changes made on ArgoCD objects through the UI or by cli tool `argocd` are reflected in updates of the ArgoCD Kubernetes objects `Application` and `AppProject` in the `{{% param argoInfraNamespace %}}` namespace.

Let's list all Kubernetes objects of type `Application` (short form: `app`)

```bash
{{% param cliToolName %}} get app --namespace={{% param argoInfraNamespace %}}
```

```
NAME               SYNC STATUS   HEALTH STATUS
argo-<username>    Synced        Healthy
```

You will see the application which we created some chapters ago by cli command `argocd app create...`. To see the complete configuration of the `Application` as _yaml_ use:

```bash
{{% param cliToolName %}} get app argo-$LAB_USER -oyaml --namespace={{% param argoInfraNamespace %}}
```

You even can edit the `Application` resource by using:

```bash
{{% param cliToolName %}} edit app argo-$LAB_USER --namespace={{% param argoInfraNamespace %}}
```

This allows us to manage the ArgoCD application definitions in a declarative way as well. It is a common pattern to have one ArgoCD application which references n child Applications which allows us a fast bootstrapping of a whole environment or a new cluster. This pattern is well known as the [App of apps](https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/#app-of-apps-pattern) pattern.


## Task {{% param sectionnumber %}}.7: Delete the Application

You can cascading delete the ArgoCD Application with the following command:

```bash
argocd app delete argo-$LAB_USER
```

This will delete the `Application` manifests of ArgoCD and all created resources by this application. In our case the `Application`, `Deployment` and `Service` will be deleted.  With the flag `--cascade=false` only the ArgoCD `Application` will be deleted and the created resources `Deployment` and `Service` remain untouched.
