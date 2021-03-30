---
title: "Labs"
weight: 2
menu:
  main:
    weight: 2
---


[Argo CD](https://argoproj.github.io/argo-cd/) ist ein Teil des [Argo Projektes](https://argoproj.github.io/) und unter der [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/) angegliedert. Das Projekt ist knapp dreijährig, komplett OpenSource und primär in Go implementiert.

Wie der Name schon verrät kümmert sich Argo CD um den continuous delivery-Aspekt von CI/CD. Die continuous integration wird durch ein CI-Tool wie GitLab CI/CD, Jenkins, Tekton oder GitHub Actions wahrgenommen. Der Core von Argo CD besteht aus einem Kubernetes-Controller, welcher kontinuierlich den live-State mit dem desired-State vergleicht. Der live-State wird dabei von der Kubernetes-API abgegriffen, und der desired-State ist in Form von Manifests in YAML oder JSON in einem Git Repository persistiert. Argo CD hilft dabei auf Abweichungen der States hinzuweisen, die Abweichungen darzustellen oder auch autonom den desired-State wiederherzustellen.

Argo CD wird auf einer Kubernetes-basierten Container Plattform deployed und betrieben. Es ist möglich mehrere Kubernetes und OpenShift Cluster an eine ArgoCD Instanz anzubinden.

{{% alert  color="primary" %}}
[Argo CD](https://argoproj.github.io/argo-cd/) is a declarative, GitOps continuous delivery tool for Kubernetes.
{{% /alert %}}

[GitOps](https://www.weave.works/technologies/gitops/) is a way to do Kubernetes cluster management and application delivery. It works by using Git as a single source of truth for declarative infrastructure and applications. With GitOps, the use of software agents can alert on any divergence between Git with what's running in a cluster, and if there's a difference, Kubernetes reconcilers automatically update or rollback the cluster depending on the case. With Git at the center of your delivery pipelines, developers use familiar tools to make pull requests to accelerate and simplify both application deployments and operations tasks to Kubernetes.

Managing Kubernetes resources using a GitOps approach brings the following benefits:

* Die Definition der Manifests erfolgt auf eine deklarative Art und ein Tool sorgt für den Abgleich zwischen den desired- und live-Manifests. Die Differenzen zwischen den gewünschten Konfigurationen und den effektiv applizierten Manifests ist jederzeit einfach ersichtlich.
* Rollbacks auf ältere Versionen sind mit git revert oder via des eingesetzten GitOps Tool einfach möglich (vorausgesetzt die Applikation unterstützt dies auch)
* Manuelle Anpassungen direkt auf der Container Plattform werden sofort ersichtlich und können automatisiert auch überschrieben werden (self-healing)
* Die git Commit History ist zugleich ein detailliertes Audit Log
* Die Entwickler beschreiben die Infrastruktur in bereits bekannten Formaten und Tools wie yaml und Git.


Argo CD follows the GitOps pattern of using Git repositories as the source of truth for defining the desired application state.

{{% alert title="Warning" color="secondary" %}}
Argo CD provides a mechanism to override the parameters of Argo CD applications. [The Argo CD parameter overrides](https://argoproj.github.io/argo-cd/user-guide/parameters/) feature is provided mainly as a convenience to developers and is intended to be used in dev/test environments, vs. production environments.

Many consider this feature as anti-pattern to GitOps. So only use this feature when no other option is available!
{{% /alert %}}

Kubernetes manifests can be specified in several ways:

* [kustomize](https://kustomize.io/) applications
* [helm](https://helm.sh/) charts
* [ksonnet](https://github.com/ksonnet/ksonnet) applications
* [jsonnet](https://jsonnet.org/) files
* Plain directory of YAML/json manifests
* Any custom config management tool configured as a config management plugin

Argo CD automates the deployment of the desired application states in the specified target environments. Application deployments can track updates to branches, tags, or pinned to a specific version of manifests at a Git commit. See tracking strategies for additional details about the different tracking strategies available.

For a quick 10 minute overview of Argo CD, check out the demo presented to the Sig Apps community meeting:

{{< youtube aWDIQMbp1cc >}}


## Argo CD Architecture

Argo CDs core components are the API Server, the Repository Server and the Application Controller

![Architecture](argocd_architecture.png)

Image and component description source: <https://argoproj.github.io/argo-cd/>


### API Server

The API server is a gRPC/REST server which exposes the API consumed by the Web UI, CLI, and CI/CD systems. It has the following responsibilities:

* application management and status reporting
* invoking of application operations (e.g. sync, rollback, user-defined actions)
* repository and cluster credential management (stored as K8s secrets)
* authentication and auth delegation to external identity providers
* RBAC enforcement
* listener/forwarder for Git webhook events


### Repository Server

The repository server is an internal service which maintains a local cache of the Git repository holding the application manifests. It is responsible for generating and returning the Kubernetes manifests when provided the following inputs:

* repository URL
* revision (commit, tag, branch)
* application path
* template specific settings: parameters, ksonnet environments, helm values.yaml


### Application Controller

The application controller is a Kubernetes controller which continuously monitors running applications and compares the current, live state against the desired target state (as specified in the repo). It detects OutOfSync application state and optionally takes corrective action. It is responsible for invoking any user-defined hooks for lifecycle events (PreSync, Sync, PostSync)


## Argo CD Core Concepts

Those core Concepts exist in Argo CD:

* Clusters: pre configured Kubernetes Clusters (including OpenShift)
* [Repositories](https://argoproj.github.io/argo-cd/user-guide/private-repositories/): pre configured git repositories, including repository credentials (ssh, username-password).
* [Applications](https://argoproj.github.io/argo-cd/operator-manual/declarative-setup/#applications): A group of Kubernetes resources, represented in a git repository. Usually the Kubernetes resources which will be applied in a Kubernetes namespace. Represented as [CRD](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/).
* [Projects](https://argoproj.github.io/argo-cd/user-guide/projects/): A logical grouping of Argo CD applications. Various restrictions can be defined on project level. Useful when multiple Teams work with the same Argo CD instance

