---
title: "Sync Windows"
weight: 82
onlyWhen: sync-windows
---

With Sync windows the user can define at which time applications can be synchronized automatically and manually by Argo CD. Allowed and forbidden time windows can be defined. Sync windows can be restricted to a subset of applications, clusters and namespaces and thus offer great flexibility.


## {{% task %}} Create application and project

Now we want to create a new empty Argo CD project.

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd proj create -s "*" -d "*,*" project-sync-windows-$USER
argocd app create sync-windows-$USER --repo https://github.com/acend/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $USER --project project-sync-windows-$USER
argocd app sync sync-windows-$USER
```

You should see the following message after a successful sync

```
...
Message:            successfully synced (all tasks run)
...
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Create `appproject.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: project-sync-windows-$USER
  namespace: {{% param argoInfraNamespace %}}
spec:
  sourceRepos:
    - '*'
  destinations:
    - server: '*'
      namespace: '*'
```

Create `application.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sync-windows-$USER
  namespace: {{% param argoInfraNamespace %}}
spec:
  project: project-sync-windows-$USER
  source:
    repoURL: https://github.com/acend/argocd-training-examples.git
    targetRevision: HEAD
    path: example-app
  destination:
    server: https://kubernetes.default.svc
    namespace: $USER
```

```bash
{{% param cliToolName %}} apply -f appproject.yaml
{{% param cliToolName %}} apply -f application.yaml
```

Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on `sync-windows-$USER`.
{{% /onlyWhen %}}


## {{% task %}} Create sync windows

Per default no sync windows are pre-configured in Argo CD. That means manual and automatic sync operations are allowed all the time. Now we want to create a sync window which denies syncs during the day between 08:00 and 20:00.

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd proj windows add project-sync-windows-$USER \
    --kind deny \
    --schedule "0 8 * * *" \
    --duration 12h \
    --applications "*" \
    --namespaces "" \
    --clusters ""
```

List all registered sync windows for the project.

```bash
argocd proj windows list project-sync-windows-$USER
```

..prints out

```
ID  STATUS  KIND  SCHEDULE   DURATION  APPLICATIONS  NAMESPACES  CLUSTERS  MANUALSYNC
0   Active  deny  0 8 * * *  12h       *             *           *         Disabled
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Update `appproject.yaml` to add a deny sync window, then re-apply:

```yaml
spec:
  # ... existing fields ...
  syncWindows:
    - kind: deny
      schedule: '0 8 * * *'
      duration: 12h
      applications:
        - '*'
      namespaces:
        - '*'
      clusters:
        - '*'
      manualSync: false
```

```bash
{{% param cliToolName %}} apply -f appproject.yaml
```
{{% /onlyWhen %}}

The window starts at 08:00 in the morning an lasts for 12 hours and denies all sync operation for all applications.

{{% alert title="Note" color="info" %}}
Paste the cron expression on [Crontab Guru](https://crontab.guru/#0_8_*_*_*) to get an explanation of it.
{{% /alert %}}

Now try to sync the previously created application

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app sync sync-windows-$USER
```

This manual sync request will be blocked due to the active sync window with the following output
```bash
FATA[0000] rpc error: code = PermissionDenied desc = Cannot sync: Blocked by sync window
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and try to click **Sync** on `sync-windows-$USER`. The sync will be blocked with a "Blocked by sync window" error.
{{% /onlyWhen %}}

{{% alert title="Note" color="info" %}}
If there is an active matching allow window and an active matching deny window then syncs will be denied as deny windows override allow windows.
{{% /alert %}}


## {{% task %}} Updating the sync window

Now we want to restrict the defined sync windows just for the application with name `sketchy-app`. We update the existing sync window with the new application name.

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd proj windows update project-sync-windows-$USER 0 --applications "sketchy-app"
```

Sync the application again
```bash
argocd app sync sync-windows-$USER
```

.. which now works because the sync window only applies for applications with the name `sketchy-app`.

Revert the changes and use wildcard `*` again to match all applications

```bash
argocd proj windows update project-sync-windows-$USER 0 --applications "*"
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Update `appproject.yaml` to restrict the sync window to `sketchy-app`, then re-apply:

```yaml
      applications:
        - 'sketchy-app'
```

```bash
{{% param cliToolName %}} apply -f appproject.yaml
```

Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on `sync-windows-$USER` — it works now because the window no longer applies to it.

Revert `applications` back to `['*']` in `appproject.yaml` and re-apply:

```bash
{{% param cliToolName %}} apply -f appproject.yaml
```
{{% /onlyWhen %}}


## {{% task %}} Enabling manual syncs

Now enable the manual sync for the window and try again to sync manually

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd proj windows enable-manual-sync project-sync-windows-$USER 0
argocd app sync sync-windows-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Update `appproject.yaml` to set `manualSync: true` on the sync window, then re-apply:

```yaml
      manualSync: true
```

```bash
{{% param cliToolName %}} apply -f appproject.yaml
```

Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on `sync-windows-$USER` — it now works for manual syncs.
{{% /onlyWhen %}}

Which now work flawless. Automatic syncs are still forbidden and will not occur between 08:00 and 20:00.


## {{% task %}} Housekeeping

Clean up the resources created in this lab

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd proj windows delete project-sync-windows-$USER 0
argocd app delete sync-windows-$USER -y
argocd proj delete project-sync-windows-$USER
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
```bash
{{% param cliToolName %}} delete application sync-windows-$USER -n {{% param argoInfraNamespace %}}
{{% param cliToolName %}} delete appproject project-sync-windows-$USER -n {{% param argoInfraNamespace %}}
```
{{% /onlyWhen %}}

Find more detailed information about [Sync Windows in the docs](https://argoproj.github.io/argo-cd/user-guide/sync_windows/#sync-windows).
