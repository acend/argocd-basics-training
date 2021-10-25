---
title: "10. Orphaned Resources"
weight: 10
sectionnumber: 10
---

This lab contains demonstrates how to find orphaned top-level resources with Argo CD. Orphaned resources are not managed by Argo CD and could be potentially removed from cluster.


## Task {{% param sectionnumber %}}.1: Create application and project

```bash
argocd app create argo-$LAB_USER --repo https://github.com/acend/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER
argocd app sync argo-$LAB_USER
```

Create new Argo CD project without restrictions for Git source repository (--src) nor destination cluster/namespace (--dest)
```bash
argocd proj create --src "*" --dest "*,*" apps-$LAB_USER
```

Enable visualization and monitoring of Orphaned Resources for the newly created project `apps-hanneloreXX`
```bash
argocd proj set apps-$LAB_USER --orphaned-resources --orphaned-resources-warn
```

{{% alert title="Note" color="primary" %}}
The flag `--orphaned-resources` enables the determinability of orphaned resources in Argo CD. After a refresh you will see them in the user interface on the project when selecting the checkbox _Orphaned Resources_.
With the flag `--orphaned-resources-warn` enabled, for each Argo CD application with orphaned resources in the destination namespace a warning will be shown in the user interface.
{{% /alert %}}


## Task {{% param sectionnumber %}}.2: Assign application to project

Assign application to newly created project
```bash
argocd app set argo-$LAB_USER --project apps-$LAB_USER
```

Ensure that the application is now assigned to the new project `apps-hanneloreXX`
```bash
argocd app get argo-$LAB_USER
```

Refresh the application
```bash
argocd app get --refresh argo-$LAB_USER
```


## Task {{% param sectionnumber %}}.3: Create orphaned resource

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

{{% alert title="Note" color="primary" %}}
This service will be detected as orphaned resource because it is not managed by Argo CD. All resources which are managed by Argo CD are marked with the label `app.kubernetes.io/instance` per default. The key of the label can be changed with the setting `application.instanceLabelKey`. See [documentation](https://argoproj.github.io/argo-cd/faq/#why-is-my-app-out-of-sync-even-after-syncing) for further details.
{{% /alert %}}


Print all resources
```bash
argocd app resources argo-$LAB_USER
```

You see in the output that the manually created service `black-hole` is marked as orphaned:
```
GROUP  KIND        NAMESPACE    NAME            ORPHANED
       Service     hannelore15  simple-example  No
apps   Deployment  hannelore15  simple-example  No
       Service     hannelore15  black-hole      Yes
```

When viewing the details of the application you will see the warning about the orphaned resource
```bash
argocd app get --refresh argo-$LAB_USER
```

```
...
CONDITION                MESSAGE                               LAST TRANSITION
OrphanedResourceWarning  Application has 1 orphaned resources  2021-09-02 16:20:36 +0200 CEST
...
```


## Task {{% param sectionnumber %}}.4: Housekeeping

Clean up the resources created in this lab

```bash
argocd app delete argo-$LAB_USER -y
argocd proj delete apps-$LAB_USER
```

Find more detailed information about [Orphaned Resources in the docs](https://argoproj.github.io/argo-cd/user-guide/orphaned-resources/).
