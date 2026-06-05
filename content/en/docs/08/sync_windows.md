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
Create `appproject-sync-window.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: project-sync-windows-<username>
  namespace: {{% param argoInfraNamespace %}}
spec:
  sourceRepos:
    - '*'
  destinations:
    - server: '*'
      namespace: '*'
```

Create `application-sync-window.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sync-windows-<username>
  namespace: {{% param argoInfraNamespace %}}
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: project-sync-windows-<username>
  source:
    repoURL: https://github.com/acend/argocd-training-examples.git
    targetRevision: HEAD
    path: example-app
  destination:
    server: https://kubernetes.default.svc
    namespace: <username>
```

```bash
{{% param cliToolName %}} apply -f appproject-sync-window.yaml
{{% param cliToolName %}} apply -f application-sync-window.yaml
```

Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on `sync-windows-<username>`.
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
Update `appproject-sync-window.yaml` to add a deny sync window, then re-apply:

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
{{% param cliToolName %}} apply -f appproject-sync-window.yaml
```
{{% /onlyWhen %}}

The window starts at 08:00 in the morning an lasts for 12 hours and denies all sync operation for all applications.

{{% alert title="Note" color="info" %}}
Paste the cron expression on [Crontab Guru](https://crontab.guru/#0_8_*_*_*) to get an explanation of it.
{{% /alert %}}

Now try to sync the previously created application.

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
Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and try to click **Sync** on `sync-windows-<username>`. The sync will be blocked with a "Blocked by sync window" error.
You will also see a red pause icon next to the Application state indicating that Sync operations are currently not allowed.
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

It still doesn't work, but why?

By default, the three fields `namespaces`, `clusters` and `applications` will be evaluated using `OR`, not `AND`. The namespace and the cluster still matches our `simple-example` application.
To change this behavior, edit the AppProject with the option `--use-and-operator`.

```bash
argocd proj windows update project-sync-windows-$USER 0 --use-and-operator
```

Try syncing again. This now works because the sync window only applies for applications with the name `sketchy-app`, the application `simple-example` is not matched anymore.
The three fields `namespaces`, `clusters` and `applications` and the functionality to toggle `OR` and `AND` evaluations give you a lot of flexibility to configure different sync windows.

Then revert the changes and use wildcard `*` again to match all applications

```bash
argocd proj windows update project-sync-windows-$USER 0 --applications "*"
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Update `appproject-sync-window.yaml` to restrict the sync window to the application `sketchy-app`, leave everything else as-is, then re-apply:

```yaml
      applications:
        - 'sketchy-app'
```

```bash
{{% param cliToolName %}} apply -f appproject-sync-window.yaml
```

Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on `sync-windows-<username>`.

It still doesn't work, but why?

By default, the three fields `namespaces`, `clusters` and `applications` will be evaluated using `OR`, not `AND`. The namespace and the cluster still matches our `simple-example` application.
To change this behavior, edit the AppProject and add the field `andOperator: true` on the same level as `manualSync`. Apply again.

```yaml
      andOperator: true
```

Try syncing again. This now works because the sync window only applies for applications with the name `sketchy-app`, the application `simple-example` is not matched anymore.
The three fields `namespaces`, `clusters` and `applications` and the functionality to toggle `OR` and `AND` evaluations give you a lot of flexibility to configure different sync windows.

Revert `applications` back to `['*']` in `appproject-sync-window.yaml` and re-apply:

```bash
{{% param cliToolName %}} apply -f appproject-sync-window.yaml
```
{{% /onlyWhen %}}


## {{% task %}} Enabling manual syncs

Now enable the manual sync for the window and try again to sync manually.

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd proj windows enable-manual-sync project-sync-windows-$USER 0
argocd app sync sync-windows-$USER
```

Which now work flawlessly.
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Update `appproject-sync-window.yaml` to set `manualSync: true` on the sync window, then re-apply:

```yaml
      manualSync: true
```

```bash
{{% param cliToolName %}} apply -f appproject-sync-window.yaml
```

Open the [Argo CD UI](https://{{% param argoCdUrl %}}) and click **Sync** on `sync-windows-<username>` — it now works for manual syncs.
Where the red pause icon was, you will now see a yellow pause icon indicating that there is an active Sync window for this app, but it does not apply for all operations.
{{% /onlyWhen %}}

Automatic syncs are still forbidden and will not occur between 08:00 and 20:00.


## {{% task %}} Enable auto-sync and prune

Enable automated sync and pruning before deletion to ensure all managed resources are cleaned up:

{{% onlyWhenNot no-argocd-cli %}}
```bash
argocd app set sync-windows-$USER --sync-policy automated
argocd app set sync-windows-$USER --self-heal
argocd app set sync-windows-$USER --auto-prune
```
{{% /onlyWhenNot %}}
{{% onlyWhen no-argocd-cli %}}
Edit `application-sync-window.yaml` to add automated sync policy, then re-apply:

```yaml
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

```bash
{{% param cliToolName %}} apply -f application-sync-window.yaml
```
{{% /onlyWhen %}}


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

Find more detailed information about [Sync Windows in the docs](https://argo-cd.readthedocs.io/en/stable/user-guide/sync_windows/).
