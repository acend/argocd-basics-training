---
title: "Simple Example"
weight: 2
onlyWhen: simple-example
---

In this lab you will learn how to deploy a simple application using Argo CD.

Our lab setup consists of the following components:

* Git Server ([Gitea](https://gitea.io)): [https://{{% param giteaUrl %}}](https://{{% param giteaUrl %}}/) -> use "sign up with dex" to sign in
* Argo CD Server: [https://{{% param argoCdUrl %}}](https://{{% param argoCdUrl %}}) -> use "login with OpenShift" to sign in
{{% onlyWhen openshift %}}
* OpenShift Cluster: [https://{{% param ocpConsoleUrl %}}](https://{{% param ocpConsoleUrl %}})
{{% /onlyWhen %}}
* The WebIDE Development Environment with all necessary tools installed will be provided by the trainer -> use "login with OpenShift" to sign in


## {{% task %}} {{% onlyWhenNot manual-fork %}}Login to the Gitea{{% /onlyWhenNot %}}{{% onlyWhen manual-fork %}}Fork the Git repository{{% /onlyWhen %}}

{{% onlyWhenNot manual-fork %}}

For this training we're using a Git Server deployed under [https://{{% param giteaUrl %}}](https://{{% param giteaUrl %}}/). We also prepared the Argo CD Example Repo for your user `<username>`.

Open your webbrowser and navigate to [https://{{% param giteaUrl %}}](https://{{% param giteaUrl %}}/).

{{% onlyWhenNot openshift %}}
Login with the training credentials provided by the trainer (Login Button is in the upper right corner).
{{% /onlyWhenNot  %}}
{{% onlyWhen openshift %}}
Click the **Sign in with dex** or **Anmelden mit dex** Button to login (Login Button is in the upper right corner).
{{% /onlyWhen%}}


{{% alert title="Note" color="info" %}}Users which have a personal Github account can just fork the Repository [argocd-training-examples](https://github.com/acend/argocd-training-examples) to their personal account. To fork the repository click on the top right of the Github on _Fork_. However, you should be aware that the ArgoCD instance is shared for all participants, meaning at some point a GitHub access token needs to be registered. Keep its permissions minimal and make sure it is disabled after this workshop. {{% /alert %}}

{{% /onlyWhenNot  %}}


{{% onlyWhen manual-fork %}}

As we are proceeding according to the GitOps principle we need some example resource manifests in a Git repository which we can edit.

Users which have a personal Github account can just fork the Repository [argocd-training-examples](https://github.com/acend/argocd-training-examples) to their personal account. To fork the repository click on the top right of the Github on _Fork_. However, you should be aware that the ArgoCD instance is shared for all participants, meaning at some point a GitHub access token needs to be registered. Keep its permissions minimal and make sure it is disabled after this workshop.

{{% alert title="Note" color="info" %}}All the cli commands in this chapter must be executed in the terminal of the provided Web IDE.{{% /alert %}}

All other users can use the provided Gitea installation of the personal lab environment.

{{% onlyWhenNot openshift %}}

Visit [https://{{% param giteaUrl %}}](https://{{% param giteaUrl %}}/) with your browser and register a new account with your personal username and a password that you can remember ;)

![Register new User in Gitea](gitea-register.png)

Login with the new user and fork the existing Git repository from Github:

{{% /onlyWhenNot%}}

{{% onlyWhen openshift %}}

Visit [https://{{% param giteaUrl %}}](https://{{% param giteaUrl %}}/) with your browser and login using **Dex** which will take you to the OpenShift login page, where you can login.

Migrate the existing Git repository from Github:

{{% /onlyWhen%}}

1. Select _Create_ on the top right -> _New Migration_ -> Select _GitHub_
1. Migrate / Clone From URL: https://github.com/acend/argocd-training-examples.git
1. Click _Migrate Repository_

{{% /onlyWhen  %}}

The Git Repository is available under your repositories.

![The Git Repository](gitea-repository.png)

By clicking on the repository link in the repository list you get to the detail page.

![The Git Repository](gitea-repository-2.png)

The **URL** of the Git repository we'll be working with will look like `https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git`.


## {{% task %}} Cloning the repository to the webshell

Within the Web IDE we set the `USER` environment variable to your personal username `<username>`.

Verify that with the following command:

```bash
echo $USER
```

The `USER` variable will be used as part of the commands to make the lab experience more comfortable for you.

{{% alert title="Note" color="info" %}}If you're **not** using our lab webshell to execute the labs, make sure to set the `USER` environment variable accordingly with the following command `export USER=<username>`{{% /alert %}}


Clone the forked repository to your local workspace:

```bash
git clone https://$USER@{{% param giteaUrl %}}/$USER/argocd-training-examples.git
```

... or the corresponding URL if you have choosen to use your own Git Server.

Change the working directory to the cloned git repository:

```bash
cd argocd-training-examples
```

When using the Web IDE: Configure the Git Client and verify the output

```bash
git config user.name "$USER"
git config user.email "$USER@{{% param giteaUrl %}}"
```

And we also want git to store our Password for the whole day so that we don't need to login every single time we push something.

```bash
git config credential.helper 'cache --timeout=86400'
```

Then use the following command to verify whether the git config for username and email were correctly added:

```bash
git config --local --list
```


## {{% task %}} Deploying the resources with Argo CD

Now we want to deploy the resource manifests contained in the cloned repository with Argo CD to demonstrate the basic features of Argo CD.

{{% onlyWhenNot no-argocd-cli %}}
To deploy the resources using the Argo CD CLI use the following command:

```bash
argocd app create argo-$USER --repo https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $USER
```

Expected output: `application 'argo-<username>' created`

{{% alert title="Note" color="info" %}}We don't need to provide Git credentials because the repository is readable for non-authenticated users as well{{% /alert %}}

{{% alert title="Note" color="info" %}}If you want to deploy it in a different namespace, make sure the namespaces exists before synching the app{{% /alert %}}

Once the application is created, you can view its status:

```bash
argocd app get argo-$USER
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
argocd app sync argo-$USER
```

This command retrieves the manifests from the git repository and performs a `{{% param cliToolName %}} apply` on them. Because all our manifests has been deployed manually before, no new rollout of them will be triggered on Kubernetes. But form now on, all resources are managed by Argo CD. Congrats, the first step in direction GitOps! :)

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

Check the [Argo CD UI](https://{{% param argoCdUrl %}}) to browse the application and their components.
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}

Create a file `example-application.yaml` in the `applications` directory with the following content:

{{% alert title="Note" color="info" %}}Make sure to replace `username` placeholders in the manifests with the correct value, if your username isn't already displayed.

Use the following command to get your username.

```bash
echo $USER
```
{{% /alert %}}

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argo-<username>
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
    targetRevision: HEAD
    path: example-app
  destination:
    server: https://kubernetes.default.svc
    namespace: <username>
```

Apply it to the cluster:

```bash
{{% param cliToolName %}} apply -f example-application.yaml
```

Expected output: `application 'argo-<username>' created`

Argo CD will now detect the application. Once the application is created, you can view its status:

```bash
{{% param cliToolName %}} describe application argo-$USER -n {{% param argoInfraNamespace %}}
```

Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** -> **Synchronize** to deploy the resources. This command retrieves the manifests from the git repository and performs a {{% param cliToolName %}} apply on them. From now on, all resources are managed by Argo CD. Congrats, the first step in direction GitOps! :)

Once synced the application status will show as **Healthy**.

```bash
{{% param cliToolName %}} get application argo-$USER -n {{% param argoInfraNamespace %}}
```
{{% /onlyWhen %}}

Application overview in unsynced and synced state

![Application overview (unsynced state)](app-overview-unsynced.png)
![Application overview (synced state)](app-overview-synced.png)

Detailed view of a application in unsynced and synced state

![Application Tree (unsynced state)](app-tree-unsynced.png)

![Application Tree (synced state)](app-tree-sycned.png)


## {{% task %}} Creating an access token for Gitea

In Gitea, click your user icon in the top right corner and click "Settings". Then go to "Applications". Here you will create an access token for Gitea. You can use "webshell" for the token
name and set the repository row to "read and write". This will be necessary for you to access your repository from the webshell.

![Creating the access token on Gitea](gitea-access-token.png)

After clicking "Generate token", it will be displayed once on top of the page. Make sure you copy the `token` somewhere you can still access later.

Optional: If you're extra security-concious, create a second token with the name "argocd" and select "read" in the repository row. We will use it later in this section.
This is optional because you can just use your webshell token for ArgoCD, but in practice you would not want write permissions where they're not needed.


## {{% task %}} Automated Sync Policy and Diff

When there is a new commit in your Git repository, the Argo CD application becomes OutOfSync. Let's assume we want to scale up our `Deployment` of the example application from 1 to 2 replicas. We will change this in the Deployment manifest.

Increase the number of replicas in your file `<workspace>/example-app/deployment.yaml` to 2.

```yaml
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
```


Commit the changes and push them to your personal remote Git repository. After the Git push command a **password** input field will appear, this is where you have to enter the toke you've created in the previous chapter.

```bash
git add example-app
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
{{% onlyWhenNot no-argocd-cli %}}

Check the state of the resources by cli:

```bash
argocd app get argo-$USER --refresh
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
argocd app diff argo-$USER
```

which should give you an output similar to:

```
===== apps/Deployment <username>/simple-example ======
101c102
<   replicas: 1
---
>   replicas: 2
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Out of the box Git will be polled by Argo CD in a predefined interval (defaults to 3 minutes). To use a synchronous workflow you can use webhooks in Git. These will trigger a synchronization in Argo CD on every push to the repository.

Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Refresh** on the `argo-<username>` application to trigger an immediate update.
{{% /onlyWhen %}}

Now open the web console of Argo CD and go to your application. The deployment `simple-example` is marked as 'OutOfSync':

![Application Out-of-Sync](app-replicas-diff-overview.png)

When an application is OutOfSync then your deployed 'live state' is no longer the same as the 'target state' which is represented by the resource manifests in the Git repository. You can inspect the differences between live and target state with a click on Deployment > Diff:

![Application Differences](app-replicas-diff-detail.png)


Now click `Sync` on the top left and let the magic happen ;) The application will be scaled up to 2 replicas and the resources are in Sync again.

{{% onlyWhenNot no-argocd-cli %}}
Double-check the status by cli

```bash
argocd app get argo-$USER
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
{{% /onlyWhenNot %}}

Argo CD can automatically sync an application when it detects differences between the desired manifests in Git, and the live state in the cluster. A benefit of automatic sync is that CI/CD pipelines no longer need direct access to the Argo CD API server to perform the deployment. Instead, the pipeline makes a commit and push to the Git repository with the changes to the manifests in the tracking Git repo.

{{% onlyWhenNot no-argocd-cli %}}
To configure automatic sync run (or use the UI):

```bash
argocd app set argo-$USER --sync-policy automated
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
To configure automatic sync, configure the `syncPolicy` and set it to `automated` by editing the `example-application.yaml` (or use the UI) a:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argo-<username>
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
    targetRevision: HEAD
    path: example-app
  destination:
    server: https://kubernetes.default.svc
    namespace: <username>
  syncPolicy:
    automated: {}
```
and re-apply the manifest:

```bash
{{% param cliToolName %}} apply -f example-application.yaml
```
{{% /onlyWhen %}}

From now on Argo CD will automatically apply all resources to Kubernetes every time you commit to the Git repository.

Decrease the replicas count to 1 and push the updated manifest to remote. Wait for a few moments and see check that ArgoCD will scale the deployment of the example app down to 1 replica. The default polling interval is 3 minutes. If you don't want to wait you can force a refresh by clicking `Refresh` in the UI{{% onlyWhenNot no-argocd-cli %}} or by cli:

```bash
argocd app get argo-$USER --refresh
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}.
{{% /onlyWhen %}}


## {{% task %}} Automatic Self-Healing

By default, changes made to the live cluster will not trigger automatic sync. To enable automatic sync when the live cluster's state deviates from the state defined in Git, {{% onlyWhenNot no-argocd-cli %}}run:

```bash
argocd app set argo-$USER --self-heal
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}edit `example-application.yaml` to set `selfHeal: true` and re-apply:

```yaml
  syncPolicy:
    automated:
      selfHeal: true
```

```bash
{{% param cliToolName %}} apply -f example-application.yaml
```
{{% /onlyWhen %}}

Watch the deployment `simple-example` in a separate terminal:

```bash
{{% param cliToolName %}} get deployment simple-example --watch --namespace=$USER
```

Let's scale our `simple-example` Deployment and observe whats happening:

```bash
{{% param cliToolName %}} scale deployment simple-example --replicas=3 --namespace=$USER
```

Argo CD will immediately scale back the `simple-example` Deployment to `1` replicas. You will see the desired replicas count in the watched Deployment.

```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
simple-example   1/1     2            2           114m
simple-example   1/3     2            2           114m
simple-example   1/3     2            2           114m
simple-example   1/3     2            2           114m
simple-example   1/3     3            2           114m
simple-example   1/1     3            2           114m
simple-example   1/1     3            2           114m
simple-example   1/1     3            2           114m
simple-example   1/1     2            2           114m
```

This is a great way to enforce a strict GitOps principle. Changes which are manually made on deployed resource manifests are reverted immediately back to the desired state by the ArgoCD controller.


## {{% task %}} Expose Application (optional)

{{% onlyWhenNot openshift %}}
To expose an application we need to specify a so called `ingress` resource. Create an `ingress.yaml` file next to the `deployment.yaml` in the example-app directory with the following content.

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-example
spec:
  rules:
    - host: simple-example-<username>.{{% param appDomain %}}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service: 
                name: simple-example
                port: 
                  number: 5000
  tls:
  - hosts:
    - simple-example-<username>.{{% param appDomain %}}
```

Commit and Push the changes again, like you did before:


```bash
git add ingress.yaml
git commit -m "Expose application"
git push

{{% /onlyWhenNot  %}}
{{% onlyWhen openshift %}}
To expose an application we need to specify a so called `route` resource. Create a `route.yaml` file next to the `deployment.yaml` in the example-app directory.

```yaml
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: simple-example
spec:
  port:
    targetPort: 5000
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
  to:
    kind: Service
    name: simple-example
    weight: 100
  wildcardPolicy: None
```

Commit and Push the changes again, like you did before:


```bash
git add example-app
git commit -m "Expose application"
git push
```
{{% /onlyWhen  %}}

After ArgoCD syncs the changes, you can access the example applications url: `https://simple-example-<username>.{{% param appDomain %}}`

Verify using the following command:

```bash
curl https://simple-example-$USER.{{% param appDomain %}}
```

The result should look similar to this:

```bash
<h1 style=color:#e81198>Hello golang</h1><h2>ID: e81198</h2>
```


## {{% task %}} Pruning

You probably asked yourself: how can I delete deployed resources on the container platform? Argo CD can be configured to delete resources that no longer exist in the Git repository.

First delete the files `service.yaml` and {{% onlyWhenNot openshift %}}`ingress.yaml`{{% /onlyWhenNot %}}{{% onlyWhen openshift %}}`route.yaml`{{% /onlyWhen %}} from Git repository and push the changes:

```bash
git add example-app
git commit -m 'Removes service and ingress' && git push

```
{{% onlyWhenNot no-argocd-cli %}}
Check the status of the application with

```bash
argocd app get argo-$USER --refresh
```

You will see that even with auto-sync and self-healing enabled the status is still OutOfSync

```
GROUP              KIND        NAMESPACE  NAME            STATUS     HEALTH   HOOK  MESSAGE
networking.k8s.io  Ingress     <username> simple-example  OutOfSync  Healthy        ingress.networking.k8s.io/simple-example created
                   Service     <username> simple-example  OutOfSync  Healthy        
apps               Deployment  <username> simple-example  Synced     Healthy
```

Now enable the auto pruning explicitly:

```bash
argocd app set argo-$USER --auto-prune
```

Recheck the status again

```bash
argocd app get argo-$USER --refresh
```

```
GROUP       KIND        NAMESPACE  NAME            STATUS     HEALTH   HOOK  MESSAGE
extensions  Ingress     <username> simple-example  Succeeded  Pruned         pruned
            Service     <username> simple-example  Succeeded  Pruned         pruned
apps        Deployment  <username> simple-example  Synced     Healthy        deployment.apps/simple-example unchanged
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Refresh** on the application. You will see that even with auto-sync enabled the resources are still OutOfSync.

To enable pruning, edit `example-application.yaml` and re-apply:

```yaml
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

```bash
{{% param cliToolName %}} apply -f example-application.yaml
```

Click **Refresh** again in the UI. The Service and Ingress/Route will now be pruned (deleted) by Argo CD.
{{% /onlyWhen %}}

The Service was successfully deleted by Argo CD because the manifest was removed from git. See the HEALTH and MESSAGE of the previous console output.


## {{% task %}} State of ArgoCD

Argo CD is largely built stateless. The configuration is persisted as native Kubernetes objects. And those are stored in Kubernetes _etcd_. There is no additional storage layer needed to run ArgoCD. The Redis storage under the hood acts just as a throw-away cache and can be evicted anytime without any data loss.

The configuration changes made on ArgoCD objects through the UI or by CLI are reflected in updates of the ArgoCD Kubernetes objects `Application` and `AppProject` in the `{{% param argoInfraNamespace %}}` namespace.

Let's list all Kubernetes objects of type `Application` (short form: `app`)

```bash
{{% param cliToolName %}} get applications --namespace={{% param argoInfraNamespace %}}
```

```
NAME               SYNC STATUS   HEALTH STATUS
argo-<username>    Synced        Healthy
```

You will see the application which we created{{% onlyWhenNot no-argocd-cli %}} some chapters ago by cli command `argocd app create...`{{% /onlyWhenNot %}}. To see the complete configuration of the `Application` as _yaml_ use:

```bash
{{% param cliToolName %}} get applications argo-$USER -oyaml --namespace={{% param argoInfraNamespace %}}
```

You can even edit the `Application` resource by using:

```bash
{{% param cliToolName %}} edit applications argo-$USER --namespace={{% param argoInfraNamespace %}}
```

This allows us to manage the ArgoCD application definitions in a declarative way as well. It is a common pattern to have one ArgoCD application which references n child Applications which allows us a fast bootstrapping of a whole environment or a new cluster. This pattern is well known as the [App of apps]({{< ref  "06" >}}) pattern.


## {{% task %}} Accessing a private Git repository

The Git repository we have imported to Gitea is publicly available for the whole world. When accessing a private repository we have to provide credentials in form of a username/password pair or a SSH private key. In this task you will learn how to access a protected repo from Argo CD.


### Step 1

First make the Git repository in Gitea **private** by checking the option `Visibility: Make Repository Private` under `Settings -> Repository`. Now refresh the app again.

{{% onlyWhenNot no-argocd-cli %}}

```bash
argocd app sync argo-$USER
```

You will see the following error
```
FATA[0000] rpc error: code = FailedPrecondition desc = authentication required
```
Argo CD can't any longer access the protected repository without providing credentials for authentication. Next assign credentials to used Git repository. You have to provide the Gitea password interactively.

```bash
argocd repo add https://{{% param giteaUrl %}}/$USER/argocd-training-examples.git --username $USER
```

{{% alert title="Note" color="info" %}}
You can provide the password through the cli by using the flag `--password`.
{{% /alert %}}

Now the sync should work. Argo CD use the configured credentials to authenticate against your repository in Gitea.

```bash
argocd app sync argo-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}

You will see an error indicating that authentication is required. There will be a popup saying `Unable to load data: revision HEAD must be resolved` and when you click on the Error in `APP CONDITIONS` it will say `Failed to load target state: failed to generate manifest for source 1 of 1: rpc error: code = Unknown desc = failed to list refs: authentication required: Unauthorized`

Argo CD can't any longer access the protected repository without providing credentials for authentication.


### Step 2

Next create a repository secret with your Gitea credentials. The password could be your actual password or an access token.

{{% alert title="Note" color="info" %}}This secret needs to be in the `argocd` namespace which is accessible by every participant in this lab. Keep the token permissions as low as possible (read on the repository suffices) and set a close expiration date!{{% /alert %}}

Create a file `repo-secret.yaml` with the following content:

{{% alert title="Note" color="info" %}}Make sure to replace  the `username` placeholders in the manifests with the correct value if it has not been replaced yet.

Use the following command to get your username.
```bash
echo $USER
```
{{% /alert %}}


```yaml
apiVersion: v1
kind: Secret
metadata:
  name: argocd-training-examples-creds-<username>
  namespace: {{% param argoInfraNamespace %}}
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: https://{{% param giteaUrl %}}/<username>/argocd-training-examples.git
  username: <username>
  password: <your-gitea-password>
```

and apply it

```bash
{{% param cliToolName %}} apply -f repo-secret.yaml
```

and sync the app again, and your Argo CD instance is able to access the private git repository using the configures secret.

Check the Argo CD UI under **Settings → Repositories** and verify the configuration of your private repository.

{{% /onlyWhen %}}


### Step 3

Finally make your personal Git repository public again for the following labs. Uncheck the option `Visibility: Make Repository Private` under `Settings -> Repository` in the Gitea UI. Remove your repository secret with the following command:

```bash
oc delete -f repo-secret.yaml
```

{{% alert title="Note" color="info" %}}
TLS certificates and SSH private keys are supported alternative authentication methods by Argo CD. Proxy support can be configured as well in the repository settings.
{{% /alert %}}

Have a look in the [documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/) for detailed information about accessing private repositories.


## {{% task %}} Delete the Application

You can cascading delete the ArgoCD Application with the following command:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app delete argo-$USER
```

Hit `y` to confirm the deletion and this will delete the `Application` manifests of ArgoCD and all created resources by this application. In our case the `Application`, `Deployment` and `Service` will be deleted.  With the flag `--cascade=false` only the ArgoCD `Application` will be deleted and the created resources `Deployment` and `Service` remain untouched.
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
```bash
{{% param cliToolName %}} delete application argo-$USER -n {{% param argoInfraNamespace %}}
```

This will delete the `Application` resource. Since automated pruning is enabled and a finalizer is set on the `Application`, Argo CD will also delete the managed `Deployment` and `Service` from the namespace.
{{% /onlyWhen %}}
