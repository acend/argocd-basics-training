---
title: "13. Application Sets"
weight: 13
sectionnumber: 13
---

{{% alert title="Note" color="primary" %}}
ArgoCD ApplicationSet ressources are in an early alpha stage!
{{% /alert %}}

With the ApplicationSet ArgoCD adds support for managing ArgoCD Application across a large number of clusters and environments. Plus it adds the capability of managing multitenant Kubernetes clusters.

ApplicationSets are defined through Custom Resource Definition and are processed by the Application set controller.

The ApplicationSet provides following features

* The ability to use a single Kubernetes manifest to target multiple Kubernetes clusters with Argo CD
* The ability to use a single Kubernetes manifest to deploy multiple applications from one or multiple Git repositories with Argo CD
* Improved support for monorepos: in the context of Argo CD, a monorepo is multiple Argo CD Application resources defined within a single Git repository
* Within multitenant clusters, improves the ability of individual cluster tenants to deploy applications using Argo CD (without needing to involve privileged cluster administrators in enabling the destination clusters/namespaces)

Here is an example of a ApplicationSet resource with a list generator and a sample guestbook application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
spec:
  generators:
  - list:
      elements:
      - cluster: engineering-dev
        url: https://1.2.3.4
      - cluster: engineering-prod
        url: https://2.4.6.8
      - cluster: finance-preprod
        url: https://9.8.7.6
  template:
    metadata:
      name: '{{cluster}}-guestbook'
    spec:
      source:
        repoURL: https://github.com/infra-team/cluster-deployments.git
        targetRevision: HEAD
        path: guestbook/{{cluster}}
      destination:
        server: '{{url}}'
        namespace: guestbook
```
The ApplicationSet resources working in a similar way as Helm templates do. You can define a set of placeholders `{{placeholder}}` which where replaced during the processing of the ApplicationSet. In the example above the playholder cluster and url are replaced with the values defined in the generator-list elements.


## Generators

There are several generators supported by the ApplicationSet controller
You can find more information about individual generators in de the [official documentation](https://argocd-applicationset.readthedocs.io/en/stable/Generators/)


### List generator

As shown in the example above, the list generator allows you to define name/URL values for clusters.


### Cluster generator

The cluster generator works the same way as the list generator. But instead of the list generator, the cluster generator determines all clusters based on the ArgoCD cluster secrets (`argocd.argoproj.io/secret-type: cluster`)

```yaml
kind: ApplicationSet
spec:
  generators:
  - clusters: {} # Automatically use all clusters defined within Argo CD
  template:
    metadata:
      name: '{{name}}-guestbook' # 'name' field of the cluster
    spec:
      source:
        # (...)
      destination:
        server: '{{server}}' # 'server' field of the cluster
        namespace: guestbook
```


### Git

The Git generator allows you to create Applications based on files within a Git repository, or based on the directory structure of a Git repository.


### Matrix

The Matrix generator may be used to combine the generated parameters of two separate generators.


## Task {{% param sectionnumber %}}.1: Create an ApplicationSet

In this lab section we're going to create an ApplicationSet for an multi-environment.
First create a ApplicationSet definition in `<workspace>/appSet.yaml` with following properties:

* Name: `example-app`
* List generator with two clusters, named `prod` and `dev`
* Url for both clusters is: `https://kubernetes.default.svc`
* As name template use: `{{cluster}}-example-app`
* Source repository: `https://github.com/acend/argocd-training-examples.git` with revision to `HEAD` and path `example-app`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: example-app
spec:
  generators:
  - list:
      elements:
      - cluster: dev
        url: https://kubernetes.default.svc
      - cluster: prod
        url: https://kubernetes.default.svc
  template:
    metadata:
      name: '{{cluster}}-example-app'
    spec:
      source:
        repoURL: https://github.com/acend/argocd-training-examples.git
        targetRevision: HEAD
        path: example-app
      destination:
        server: https://kubernetes.default.svc
        namespace: <LAB_USER>
```
