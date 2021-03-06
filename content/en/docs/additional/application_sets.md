---
title: "9.3 Application Sets"
weight: 903
sectionnumber: 9.3
---

{{% alert title="Note" color="primary" %}}
ArgoCD ApplicationSet ressources are in an early alpha stage!
If you want to try the application set by yourself, make sure that the [ApplicationSet Controller is installed](https://argocd-applicationset.readthedocs.io/en/stable/Getting-Started/#a-install-applicationset-into-an-existing-argo-cd-install) alongside the ArgoCD controller.
{{% /alert %}}

With the ApplicationSet ArgoCD adds support for managing ArgoCD Application across a large number of clusters and environments. Plus it adds the capability of managing multitenant Kubernetes clusters.

ApplicationSets are defined through Custom Resource Definition and are processed by the Application set controller.

The ApplicationSet provides following features

* The ability to use a single Kubernetes manifest to target multiple Kubernetes clusters with Argo CD
* The ability to use a single Kubernetes manifest to deploy multiple applications from one or multiple Git repositories with Argo CD
* Improved support for monorepos: in the context of Argo CD, a monorepo is multiple Argo CD Application resources defined within a single Git repository
* Within multitenant clusters, improves the ability of individual cluster tenants to deploy applications using Argo CD (without needing to involve privileged cluster administrators in enabling the destination clusters/namespaces)

Here is a graphical overview how ArgoCD process an ApplicationSet.
![How ApplicationSet are processed](../applicationSet-overview.png)

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

The Matrix generator combines the parameters generated by two child generators, iterating through every combination of each generator's generated parameters.

* **SCM Provider Generator + Cluster Generator:** Scanning the repositories of a GitHub organization for application resources, and targeting those resources to all available clusters.
* **Git File Generator + List Generator:** Providing a list of applicatations to deploy via configuration files, with optional configuration options, and deploying them to a fixed list of clusters.
* **Git Directory Generator + Cluster Decision Resource Generator:** Locate application resources contained within folders of a Git repository, and deploy them to a list of clusters provided via an external custom resource.
* And so on...


#### Example

Example of a matrix generator which combines a Git generator and a cluster generator.
Assume that we have two clusters defined in our secrets.
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-git
spec:
  generators:
    # matrix 'parent' generator
    - matrix:
        generators:
          # git generator, 'child' #1
          - git:
              repoURL: https://github.com/argoproj-labs/applicationset.git
              revision: HEAD
              directories:
                - path: examples/matrix/cluster-addons/*
          # cluster generator, 'child' #2
          - clusters:
              selector:
                matchLabels:
                  argocd.argoproj.io/secret-type: cluster
  template:
    metadata:
      name: '{{path.basename}}-{{name}}'
    spec:
      project: '{{metadata.labels.environment}}'
      source:
        repoURL: https://github.com/argoproj-labs/applicationset.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: '{{server}}'
        namespace: '{{path.basename}}'
```

The git generator generate following output:
```yaml
- path: /examples/git-generator-directory/cluster-addons/argo-workflows
  path.basename: argo-workflows

- path: /examples/git-generator-directory/cluster-addons/prometheus-operator
  path.basename: prometheus-operator
```

The cluster generator generate following output:
```yaml
- name: staging
  server: https://1.2.3.4

- name: production
  server: https://2.4.6.8
```


The combined output is as follow
```yaml
- name: staging
  server: https://1.2.3.4
  path: /examples/git-generator-directory/cluster-addons/argo-workflows
  path.basename: argo-workflows

- name: staging
  server: https://1.2.3.4
  path: /examples/git-generator-directory/cluster-addons/prometheus-operator
  path.basename: prometheus-operator

- name: production
  server: https://2.4.6.8
  path: /examples/git-generator-directory/cluster-addons/argo-workflows
  path.basename: argo-workflows

- name: production
  server: https://2.4.6.8
  path: /examples/git-generator-directory/cluster-addons/prometheus-operator
  path.basename: prometheus-operator
```


## Task {{% param sectionnumber %}}.1: Create an ApplicationSet

In this lab section we're going to create an ApplicationSet for an multi-environment.
First create a ApplicationSet definition in `<workspace>/appSet.yaml` with following properties:

* Name: `example-app`
* Matrix generator with two child generators
  * Git directory generator pointing on `applications/*`
  * List generator with two clusters, named `prod` and `dev`, both with the default Kubernetes API url `https://kubernetes.default.svc`
* As name template use: `as-{{path.basename}}-{{cluster}}`
* Source repository: `https://github.com/schlapzz/argocd-applicationset.git` with revision to `HEAD` and path `{{path}}`
* For the namespace use `<LAB_USER>-{{cluster}}`

In the end the matrix generator should produce in total 4 Applications with following properties:

```yaml
- name: as-producer-dev
  path: applications/producer/
  path.basename: producer
  namespace: <LAB_USER>-dev

- name: as-consumer-prod
  path: applications/consumer/
  path.basename: consumer
  namespace: <LAB_USER>-prod

- name: as-producer-dev
  path: applications/producer/
  namespace: <LAB_USER>-dev

- name: as-consumer-prod
  path: applications/consumer/
  path.basename: consumer
  namespace: <LAB_USER>-prod
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: application-set-example
spec:
  generators:
    - matrix:
        generators:
          - list:
              elements:
              - cluster: dev
                url: https://kubernetes.default.svc
              - cluster: prod
                url: https://kubernetes.default.svc
          - git:
              repoURL: https://github.com/schlapzz/argocd-applicationset.git
              revision: HEAD
              directories:
                - path: applications/*
  template:
    metadata:
      name: 'as-{{path.basename}}-{{cluster}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/schlapzz/argocd-applicationset.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '<LAB_USER>-{{cluster}}'
```

Next check the ArgoCD web ui, you should see the 4 generator and deployed ArgoCD applications.

![Application generator from ApplicationSet](../applicationSet-ui.png)
