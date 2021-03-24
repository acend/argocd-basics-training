---
title: "2. Simple Example"
weight: 2
sectionnumber: 2
---


## Task {{% param sectionnumber %}}.1: Getting started

You can access Argo CD via Web UI (URL is provided by your teacher) or using the CLI. The Argo CD CLI Tool is already installed on the web IDE.

Since the sso login does not work inside the Web IDE for various reasons, your teacher will provide a generic local Argo CD account `hannelore` without any number.

```bash
argocd login {{% param argoCdUrl %}} --grpc-web --username hannelore
```

{{% alert title="Note" color="primary" %}}Make sure to pass the `<ARGOCD_SERVER>` without protocol e.g. `argocd.domain.com`. The `--grpc-web` parameter is necessary due to missing http 2.0 router.{{% /alert %}}


## Task {{% param sectionnumber %}}.2: Fork the Git repository

As we are proceeding according to the GitOps principle we need some example resource manifests in a Git repository which we can edit. 

Users which have a personal Github account can just fork the Repository [amm-argocd-example](https://github.com/puzzle/amm-argocd-example) to their personal account. To fork the repository click on the top right of the Github on _Fork_.

All other users can use the provided Gitea installation of the personal lab environment. Visit `https://{{% param giteaUrl %}}/` with your browser and register a new account with your personal username and a password that you can remember ;)

{{% alert title="Note" color="primary" %}}All the cli commands in this chapter must be executed in the terminal of the provided Web IDE.{{% /alert %}}

![Register new User in Gitea](gitea-register.png)

Login with the new user and fork the existing Git repository from Github:

1. Select _Create_ on the top right -> _New Migration_ -> Select _GitHub_
1. Migrate / Clone From URL: https://github.com/puzzle/amm-argocd-example.git
1. Click _Migrate Repository_
 
The URL of the newly forked Git repository will look like `https://{{% param giteaUrl %}}/<username>/amm-argocd-example.git`

Set the `LAB_USER` environment variable to your personal user:

```bash
export LAB_USER=<username>
echo $LAB_USER
```

When using the Web IDE: Configure the Git Client and verify the output

```bash
git config user.name "$LAB_USER"
git config user.email "foo@bar.org"
git config --local --list
```

Clone the forked repository to your local workspace:

```bash
git clone https://$LAB_USER@{{% param giteaUrl %}}/$LAB_USER/amm-argocd-example.git
```

... or when forked to Github with:

```bash
https://github.com/<github-username>/amm-argocd-example
```

Change the working directory to the cloned git repository: `cd amm-argocd-example`


## Task {{% param sectionnumber %}}.3: Deploying the resources with Argo CD

Now we want to deploy the resource manifests contained in the cloned repository with Argo CD to demonstrate the basic features of Argo CD.

Change to your main Namespace.

```bash
kubectl config set-context --current --namespace=$LAB_USER
```

To deploy the resources using the Argo CD CLI use the following command:

```bash
argocd app create argo-$LAB_USER --repo https://{{% param giteaUrl %}}/$LAB_USER/amm-argocd-example.git --path 'example-app' --dest-server https://kubernetes.default.svc --project  --dest-namespace $LAB_USER
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
Repo:               https://{{% param giteaUrl %}}/<username>/amm-argocd-example.git
Target:             
Path:               example-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (5a6f365)
Health Status:      Missing

GROUP  KIND        NAMESPACE    NAME                           STATUS     HEALTH   HOOK  MESSAGE
       Service     <username>  example-php-docker-helloworld  OutOfSync  Missing
apps   Deployment  <username>  example-php-docker-helloworld  OutOfSync  Missing
```


The application status is initially in OutOfSync state. To sync (deploy) the resource manifests, run:

```bash
argocd app sync argo-$LAB_USER
```

This command retrieves the manifests from the git repository and performs a `kubectl apply` on them. Because all our manifests has been deployed manually before, no new rollout of them will be triggered on OpenShift. But form now on, all resources are managed by Argo CD. Congrats, the first step in direction GitOps! :)

You should see an output similar to the following lines:

```
TIMESTAMP                  GROUP        KIND   NAMESPACE                   NAME             STATUS    HEALTH        HOOK  MESSAGE
2021-03-24T14:19:16+01:00            Service  hannelore42  example-php-docker-helloworld  OutOfSync  Missing
2021-03-24T14:19:16+01:00   apps  Deployment  hannelore42  example-php-docker-helloworld  OutOfSync  Missing
2021-03-24T14:19:16+01:00            Service  hannelore42  example-php-docker-helloworld    Synced  Healthy

Name:               argo-<username>
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          <username>
URL:                https://{{% param argoCdUrl %}}/applications/argo-<username>
Repo:               https://{{% param giteaUrl %}}/<username>/amm-argocd-example.git
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

GROUP  KIND        NAMESPACE    NAME                           STATUS  HEALTH       HOOK  MESSAGE
       Service     <username>  example-php-docker-helloworld  Synced  Healthy            service/example-php-docker-helloworld created
apps   Deployment  <username>  example-php-docker-helloworld  Synced  Progressing        deployment.apps/example-php-docker-helloworld created
```

Check the Argo CD UI to browse the application and their components. The URL of the Argo CD webinterface will be provided by the teacher.

Application overview in unsynced and synced state
<img src="app-overview-unsynced.png" alt="drawing" width="300"/>
<img src="app-overview-synced.png" alt="drawing" width="300"/>

Detailed view of a application in unsynced and synced state
![Application Tree (unsynced state)](app-tree-unsynced.png)

![Application Tree (synced state)](app-tree-sycned.png)


## Task {{% param sectionnumber %}}.4: Automated Sync Policy and Diff

When there is a new commit in your Git repository, the Argo CD application becomes OutOfSync. Let's assume we want to scale up our producer of the previous lab from 1 to 2 replicas. We will change this in the Deployment.


Change the number of replicas in your file `<workspace>/producer.yaml`.

```
{{< highlight YAML "hl_lines=9" >}}
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: data-producer
    application: amm-techlab
  name: data-producer
spec:
  replicas: 2
  selector:
    matchLabels:
      deployment: data-producer
  strategy:
    type: Recreate
...
{{< / highlight >}}
```

Commit the changes and push them to the remote:





// TODO: describe initial authentication against git

Now add the resource definitions to your personal Git repository and push them to remote. Use the password you entered when creating your Gitea user.

After the Git push command a password input field will appear at the top of the Web IDE. You need to enter your Gitea password there.

```bash
git add --all
git commit -m "Initial commit of resource definitions"
git remote add origin https://$LAB_USER@{{% param giteaUrl %}}/$LAB_USER/amm-argocd-example.git
git push -u origin master

```

After a successful push you should see the following output

```bash
Enumerating objects: 15, done.
Counting objects: 100% (15/15), done.
Delta compression using up to 4 threads
Compressing objects: 100% (15/15), done.
Writing objects: 100% (15/15), 4.02 KiB | 4.02 MiB/s, done.
Total 15 (delta 1), reused 0 (delta 0)
remote: . Processing 1 references
remote: Processed 1 references in total
To https://{{% param giteaUrl %}}/<username>/amm-argocd-example.git
 * [new branch]      master -> master
```

Go back to the webinterface of Gitea and inspect the structure and files in your personal Git repository: `https://{{% param giteaUrl %}}/<username>/amm-argocd-example`

// TODO ends





```bash
git add . && git commit -m 'Scaled up to 2 replicas' && git push
```

Don't forget to interactively provide your personal Git password. After a successful push you should see a message similar to the following lines:

```
[master 18daed3] Scaled up to 2 replicas
 1 file changed, 1 insertion(+), 1 deletion(-)
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 372 bytes | 372.00 KiB/s, done.
Total 4 (delta 2), reused 0 (delta 0)
remote: . Processing 1 references
remote: Processed 1 references in total
To https://{{% param giteaUrl %}}/<username>/amm-argocd-example.git
   fe4e2b6..18daed3  master -> master
```


Check the state of the resources by cli:

```bash
argocd app get argo-$LAB_USER --refresh
```

The parameter `--refresh` triggers an update against the Git repository. Out of the box Git will be polled by Argo CD. To use a synchronous workflow you can use webhooks in Git. These will trigger a synchronization in Argo CD on every push to the repository.

You will see that the data-producer is OutOfSync:

```
...
GROUP               KIND         NAMESPACE    NAME           STATUS     HEALTH   HOOK  MESSAGE
                    Service      <username>  data-producer  Synced     Healthy        service/data-producer unchanged
                    Service      <username>  data-consumer  Synced     Healthy        service/data-consumer unchanged
apps                Deployment   <username>  data-producer  OutOfSync  Healthy        deployment.apps/data-producer configured
apps                Deployment   <username>  data-consumer  Synced     Healthy        deployment.apps/data-consumer unchanged
...
```

When an application is OutOfSync then your deployed 'live state' is no longer the same as the 'target state' which is represented by the resource manifests in the Git repository. You can inspect the differences between live and target state by cli:

```bash
argocd app diff argo-$LAB_USER
```

which should give you an output similar to:

```
===== apps/Deployment <username>/data-producer ======
155c155
<   replicas: 1
---
>   replicas: 2
```

Now open the web console of Argo CD and go to your application. The deployment `data-producer` is marked as 'OutOfSync':

![Application Out-of-Sync](../argo-outofsynch.png)

With a click on Deployment > Diff you will see the differences:

![Application Differences](../argo-diff.png)


Now click `Sync` on the top left and let the magic happens ;) The producer will be scaled up to 2 replicas and the resources are in Sync again.

Double-check the status by cli

```bash
argocd app get argo-$LAB_USER
```

```
...
GROUP               KIND         NAMESPACE    NAME           STATUS  HEALTH       HOOK  MESSAGE
                    Service      <username>  data-consumer  Synced  Healthy            service/data-consumer unchanged
                    Service      <username>  data-producer  Synced  Healthy            service/data-producer unchanged
apps                Deployment   <username>  data-consumer  Synced  Healthy            deployment.apps/data-consumer unchanged
apps                Deployment   <username>  data-producer  Synced  Progressing        deployment.apps/data-producer configured
kafka.strimzi.io    Kafka        <username>  amm-techlab    Synced                     kafka.kafka.strimzi.io/amm-techlab unchanged
...
```

Argo CD can automatically sync an application when it detects differences between the desired manifests in Git, and the live state in the cluster. A benefit of automatic sync is that CI/CD pipelines no longer need direct access to the Argo CD API server to perform the deployment. Instead, the pipeline makes a commit and push to the Git repository with the changes to the manifests in the tracking Git repo.

To configure automatic sync run (or use the UI):

```bash
argocd app set argo-$LAB_USER --sync-policy automated
```

From now on Argo CD will automatically synchronize resources every time you commit to the Git repository.


## Task {{% param sectionnumber %}}.5: Automatic Self-Healing

By default, changes made to the live cluster will not trigger automatic sync. To enable automatic sync when the live cluster's state deviates from the state defined in Git, run:

```bash
argocd app set argo-$LAB_USER --self-heal
```

Watch the deployment `data-producer` in a separate terminal

```bash
kubectl get deployment data-producer -w
```

Let's scale our `data-producer` Deployment and observe whats happening:

```bash
kubectl scale deployment data-producer --replicas=1
```

Argo CD will immediately scale back the `data-producer` Deployment to `2` replicas. You will see the desired replicas count in the watched Deployment.

```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
data-producer   2/2     2            2           78m
data-producer   2/1     2            2           78m
data-producer   2/1     2            2           78m
data-producer   1/1     1            1           78m
data-producer   1/2     1            1           78m
data-producer   1/2     1            1           78m
data-producer   1/2     1            1           78m
data-producer   1/2     2            1           78m
data-producer   2/2     2            2           78m
```


## Task {{% param sectionnumber %}}.7: Pruning

You probably asked yourself how can I delete deployed resources on the container platform? Argo CD can be configured to delete resources that no longer exist in the Git repository.

First delete the file `imageStream.yaml` from Git repository and push the changes

```bash
git rm imageStream.yaml
git add --all && git commit -m 'Removes ImageStream' && git push

```

Check the status of the application with

```bash
argocd app get argo-$LAB_USER --refresh
```

You will see that even with auto-sync and self-healing enabled the status is still OutOfSync

```
GROUP               KIND         NAMESPACE    NAME           STATUS     HEALTH   HOOK  MESSAGE
...
build.openshift.io  BuildConfig  <username>  data-producer  Synced
image.openshift.io  ImageStream  <username>  data-producer  OutOfSync
kafka.strimzi.io    Kafka        <username>  amm-techlab    Synced
...
```

Now enable the auto pruning explicitly:

```bash
argocd app set argo-$LAB_USER --auto-prune
```

Recheck the status again

```bash
argocd app get argo-$LAB_USER --refresh
```

Now the ImageStream was successfully deleted by Argo CD.

```
GROUP               KIND         NAMESPACE    NAME           STATUS     HEALTH   HOOK  MESSAGE
...
image.openshift.io  ImageStream  <username>  data-producer  Succeeded  Pruned         pruned
                    Service      <username>  data-producer  Synced     Healthy        service/data-producer unchanged
                    Service      <username>  data-consumer  Synced     Healthy        service/data-consumer unchanged
apps                Deployment   <username>  data-producer  Synced     Healthy        deployment.apps/data-producer unchanged
...

```


## Task {{% param sectionnumber %}}.8: State of ArgoCD

Argo CD is largely built stateless. The configuration is persisted as native Kubernetes objects. And those are stored in Kubernetes _etcd_. There is no additional storage layer needed to run ArgoCD. The Redis storage under the hood acts just as a throw-away cache and can be evicted anytime without any data loss.

The configuration changes made on ArgoCD objects through the UI or by cli tool `argocd` are reflected in updates of the ArgoCD Kubernetes objects `Application` and `AppProject`.

Let's list all Kubernetes objects of type `Application` (short form: `app`)

```bash
kubectl get app
```

```
NAME               SYNC STATUS   HEALTH STATUS
argo-<username>    Synced        Healthy
```

You will see the application which we created some chapters ago by cli command `argocd app create...`. To see the complete configuration of the `Application` as _yaml_ use:

```bash
kubectl get app argo-<username> -oyaml
```

You even can edit the `Application` resource by using:

```bash
kubectl edit app argo-<username>
```

This allows us to manage the ArgoCD application definitions in a declarative way as well. It is a common pattern to have one ArgoCD application which references n child Applications which allows us a fast bootstrapping of a whole environment or a new cluster. This pattern is well known as the [App of apps](https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/#app-of-apps-pattern) pattern. 


## Task {{% param sectionnumber %}}.9: Delete the Application

You can cascading delete the ArgoCD Application with the following command:

```bash
argocd app delete argo-$LAB_USER
```

This will delete the `Application` Manifests of ArgoCD and all created resources by this application. In our case the `Application`, `Deployment` and `Service` will be deleted.  With the flag `--cascade=false` only the ArgoCD `Application` will be deleted and the created resources `Deployment` and `Service` remain untouched.
