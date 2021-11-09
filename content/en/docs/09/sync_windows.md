---
title: "9.2 Sync Windows"
weight: 92
sectionnumber: 9.2
---

With Sync windows the user can define at which time applications can be synchronized automatically and manually by Argo CD. Allowed and forbidden time windows can be defined. Sync windows can be restricted to a subset of applications, clusters and namespaces and thus offer great flexibility.


## Task {{% param sectionnumber %}}.1: Create application and project

Now we want to create a new empty Argo CD project.

```bash
argocd proj create -s "*" -d "*,*" project-sync-windows-$LAB_USER
argocd app create sync-windows-$LAB_USER --repo https://github.com/acend/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER --project project-sync-windows-$LAB_USER
argocd app sync sync-windows-$LAB_USER
```

You should see the following message after a successful sync

```
...
Message:            successfully synced (all tasks run)
...
```


## Task {{% param sectionnumber %}}.2: Create sync windows

Per default no sync windows are pre-configured in Argo CD. That means manual and automatic sync operations are allowed all the time. Now we want to create a sync window which denies syncs during the day between 08:00 and 20:00.


```bash
argocd proj windows add project-sync-windows-$LAB_USER \
    --kind deny \
    --schedule "0 8 * * *" \
    --duration 12h \
    --applications "*" \
    --namespaces "" \
    --clusters ""
```

List all registered sync windows for the project.

```bash
argocd proj windows list project-sync-windows-$LAB_USER
```

..prints out

```
ID  STATUS  KIND  SCHEDULE   DURATION  APPLICATIONS  NAMESPACES  CLUSTERS  MANUALSYNC
0   Active  deny  0 8 * * *  12h       *             *           *         Disabled
```
The window starts at 08:00 in the morning an lasts for 12 hours and denies all sync operation for all applications.

{{% alert title="Note" color="primary" %}}
Paste the cron expression on [Crontab Guru](https://crontab.guru/#0_8_*_*_*) to get an explanation of it.
{{% /alert %}}

Now try to sync the previously created application

```bash
argocd app sync sync-windows-$LAB_USER
```

This manual sync request will be blocked due to the active sync window with the following output
```bash
FATA[0000] rpc error: code = PermissionDenied desc = Cannot sync: Blocked by sync window
```

{{% alert title="Note" color="primary" %}}
If there is an active matching allow window and an active matching deny window then syncs will be denied as deny windows override allow windows.
{{% /alert %}}


## Task {{% param sectionnumber %}}.3: Updating the sync window

Now we want to restrict the defined sync windows just for the application with name `sketchy-app`. We update the existing sync window with the new application name.

```bash
argocd proj windows update project-sync-windows-$LAB_USER 0 --applications "sketchy-app"
```

Sync the application again
```bash
argocd app sync sync-windows-$LAB_USER
```

.. which now works because the sync window only applies for applications with the name `sketchy-app`.

Revert the changes and use wildcard `*` again to match all applications

```bash
argocd proj windows update project-sync-windows-$LAB_USER 0 --applications "*"
```


## Task {{% param sectionnumber %}}.4: Enabling manual syncs

Now enable the manual sync for the window and try again to sync manually

```bash
argocd proj windows enable-manual-sync project-sync-windows-$LAB_USER 0
argocd app sync sync-windows-$LAB_USER
```

Which now work flawless. Automatic syncs are still forbidden and will not occur between 08:00 and 20:00.


## Task {{% param sectionnumber %}}.5: Housekeeping

Clean up the resources created in this lab

```bash
argocd proj windows delete project-sync-windows-$LAB_USER 0
argocd app delete sync-windows-$LAB_USER -y
argocd proj delete project-sync-windows-$LAB_USER
```

Find more detailed information about [Sync Windows in the docs](https://argoproj.github.io/argo-cd/user-guide/sync_windows/#sync-windows).
