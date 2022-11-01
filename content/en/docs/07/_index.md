---
title: "7. Projects"
weight: 7
sectionnumber: 7
---

Argo CD applications can be linked to a project which provides a logical grouping of applications. The following configurations can be made on a project:

* Source repositories: Git repositories where application manifests are permitted to be retrieved from
* Destination Clusters: Permitted destination Kubernetes or OpenShift clusters
* Destination Namespaces: Destination namespaces where the manifests are permitted to be deployed to
* Permitted resource kinds to be synced (e.g. `ConfigMap`)
* Sync windows: Time windows when an application is permitted to be synced by Argo CD.
* Roles: Roles and policies assigned to the project. The roles are bound to OIDC groups and/or JWT tokens)
* GPG Signature Keys: GnuPG keys that commits must be signed with in order to be allowed to sync them
* Resource Monitoring: Visualization and monitoring of orphaned resources

In summary, a project defines who can deploy what to which destination. This is very useful to keep the isolation between different user groups working on the same Argo CD instance and enables the capability of multi tenancy.


## Task {{% param sectionnumber %}}.1: Create a new empty project

Now we want to create a new empty Argo CD project.

```bash
argocd proj create project-$STUDENT
argocd proj list
```

You should find in the output your newly created project:

```
NAME                 DESCRIPTION  DESTINATIONS  SOURCES  CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  SIGNATURE-KEYS  ORPHANED-RESOURCES
...
default                           *,*           *        */*                         <none>                        <none>          disabled
project-<username>               <none>        <none>   <none>                      <none>                        <none>          disabled
...
```


## Task {{% param sectionnumber %}}.2: Define permitted sources and destinations

The next step is to deploy a new application and assign it to the created project `project-<username>` by using the flag `--project`

```bash
argocd app create project-app-$STUDENT --repo https://github.com/acend/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $STUDENT --project project-$STUDENT
```

You will receive an error when trying to create the new application
```
FATA[0000] rpc error: code = InvalidArgument desc = application spec is invalid: InvalidSpecError: application repo https://github.com/acend/argocd-training-examples.git is not permitted in project 'project-<username>';InvalidSpecError: application destination {https://kubernetes.default.svc <username>} is not permitted in project 'project-<username>'
```

The cause for this error is the missing setting for the allowed destination clusters and namespaces on the project. We will fix that by setting the allowed destination cluster to `https://kubernetes.default.svc` and using the wildcard expression `user*` as allowed namespace names.

```bash
argocd proj add-destination project-$STUDENT https://kubernetes.default.svc "user*"
```

The same issue would happen because of the missing source repository expression. We will use the wildcard "*" to allow all source repositories.

```bash
argocd proj add-source project-$STUDENT "*"
```

Now print out the details of the project again

```bash
argocd proj get project-$STUDENT
```

... and you will see the permitted source repository and destination cluster/namespace:

```
...
Destinations:                https://kubernetes.default.svc,user*
Repositories:                *
...
```

Now you should be able to create a new application linked with the project

```bash
argocd app create project-app-$STUDENT --repo https://github.com/acend/argocd-training-examples.git --path 'example-app' --dest-server https://kubernetes.default.svc --dest-namespace $STUDENT --project project-$STUDENT
```

Now sync the application manifest

```bash
argocd app sync project-app-$STUDENT
```

{{% alert title="Note" color="primary" %}}
The feature of limiting source repositories and destination clusters/namespaces is a powerful construct of Argo CD as roles and policies can be assigned to projects. With this tool you can enforce a fine grained permission model to control the access of the users to the different clusters and namespaces.
{{% /alert %}}


## Task {{% param sectionnumber %}}.3: Deny resources by kind

On a project there is the possibility to restrict the kind of resources that can be synchronized. The restrictions are defined by whitelisting for cluster scoped resources and blacklisted for namespace scoped resources.

Let's extend our existing project and deny the synchronization of `Services`.

```bash
argocd proj deny-namespace-resource project-$STUDENT "" Service
```

Now sync the application
```bash
argocd app sync project-app-$STUDENT
```

The sync operation will fail with the following error

```
...
GROUP  KIND        NAMESPACE    NAME            STATUS   HEALTH   HOOK  MESSAGE
       Service     <username>   simple-example  Unknown  Missing        Resource :Service is not permitted in project project-<username>.
apps   Deployment  <username>   simple-example  Synced   Healthy
FATA[0001] Operation has completed with phase: Failed
```

Remove the kind `Service` from the deny list by using `allow-namespace-resource`

```bash
argocd proj allow-namespace-resource project-$STUDENT "" Service
```

... and sync the app again
```bash
argocd app sync project-app-$STUDENT
```


## Task {{% param sectionnumber %}}.4: Cleanup

Delete the resources created in this chapter by running the following commands:

```bash
argocd app delete project-app-$STUDENT
argocd proj delete project-$STUDENT
```
