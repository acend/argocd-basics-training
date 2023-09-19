---
title: "9.2 Backup and Restore"
weight: 902
sectionnumber: 9.2
---

{{% alert title="Warning" color="secondary" %}}
This Lab only works on your local machine.
{{% /alert %}}


## Backup and Restore

As ArgoCD holds the whole state in native Kubernetes objects it's quite straightforward to make a backup and restore it in case of a disaster recovery.

ArgoCD provides the utility [argocd-util](https://argoproj.github.io/argo-cd/operator-manual/server-commands/argocd-util) which is used by ArgoCD internally. The functions [export](https://argoproj.github.io/argo-cd/operator-manual/server-commands/argocd-util_export) and [import](https://argoproj.github.io/argo-cd/operator-manual/server-commands/argocd-util_import) can be used for backup and restore of a ArgoCD instance.

As the tool is contained inside the ArgoCD server image you just can execute it from inside the container.

First find the current version used of ArgoCD server to align the server version with the tool version:

```bash
$ argocd version | grep server

argocd-server: v2.6.3+e05298b
```

Export the version (without the postfix +eb3d1fb) to environment:

```bash
export VERSION=v2.6.3
```


### Backup

```bash
docker run -v ~/.kube:/home/argocd/.kube --rm argoproj/argocd:$VERSION argocd admin export > backup.yaml
```

{{% alert title="Note" color="primary" %}}
If you should encounter permission errors like `error loading config file \"/home/argocd/.kube/config\": open /home/argocd/.kube/config: permission denied` you should change temporarily the permission of the kube config:

```bash
chmod o+r ~/.kube/config
```

Don't forget to remove the read permission for others after exporting:
```bash
chmod o-r ~/.kube/config
```
{{% /alert %}}


### Restore

The same utility can be used to restore a previously made backup:

```bash
docker run -i -v ~/.kube:/home/argocd/.kube --rm argoproj/argocd:$VERSION argocd admin import - < backup.yaml
```

Reference: [Disaster recovery](https://argoproj.github.io/argo-cd/operator-manual/disaster_recovery/)
