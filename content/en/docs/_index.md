---
title: "Labs"
weight: 2
menu:
  main:
    weight: 2
---


[Argo CD](https://argoproj.github.io/argo-cd/) is a part of the [Argo Project](https://argoproj.github.io/) and affiliated under the [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/). The project is just under three years old, completely open source, and primarily implemented in Go.

As the name suggests, Argo CD takes care of the continuous delivery aspect of CI/CD. Continuous integration is handled by a CI tool such as GitLab CI/CD, Jenkins, Tekton or GitHub Actions. The core of Argo CD consists of a Kubernetes controller, which continuously compares the live-state with the desired-state. The live-state is tapped from the Kubernetes API, and the desired-state is persisted in the form of manifests in YAML or JSON in a Git repository. Argo CD helps to point out deviations of the states, to display the deviations or to autonomously restore the desired state.

Argo CD is deployed and operated on a Kubernetes-based container platform. It is possible to connect multiple Kubernetes and OpenShift clusters to one Argo CD instance.

{{% alert  color="info" %}}
[Argo CD](https://argoproj.github.io/argo-cd/) is a declarative, GitOps continuous delivery tool for Kubernetes.
{{% /alert %}}

[GitOps](https://www.weave.works/technologies/gitops/) is a way to do Kubernetes cluster management and application delivery. It works by using Git as a single source of truth for declarative infrastructure and applications. With GitOps, the use of software agents can alert on any divergence between Git with what's running in a cluster, and if there's a difference, Kubernetes reconcilers automatically update or rollback the cluster depending on the case. With Git at the center of your delivery pipelines, developers use familiar tools to make pull requests to accelerate and simplify both application deployments and operations tasks to Kubernetes.

Managing Kubernetes resources using a GitOps approach brings the following benefits:

* The definition of the manifests is done in a declarative way and a tool ensures the comparison between the desired and live manifests. The differences between the desired configurations and the actually applied manifests can be easily seen at any time.
* Rollbacks to older versions are easily possible with git revert or via the used GitOps tool (provided that the application also supports this).
* Manual adjustments directly on the container platform are immediately visible and can be automatically overwritten (self-healing).
* The git commit history is also a detailed audit log
* The developers describe the infrastructure in already known formats and tools like yaml and git.

Argo CD follows the GitOps pattern of using Git repositories as the source of truth for defining the desired application state.

{{% alert title="Warning" color="warning" %}}
Argo CD provides a mechanism to override the parameters of Argo CD applications. [The Argo CD parameter overrides](https://argoproj.github.io/argo-cd/user-guide/parameters/) feature is provided mainly as a convenience to developers and is intended to be used in dev/test environments, vs. production environments.

Many consider this feature as anti-pattern to GitOps. So only use this feature when no other option is available!
{{% /alert %}}

Kubernetes manifests can be specified in several ways:

* [kustomize](https://kustomize.io/) applications
* [helm](https://helm.sh/) charts
* [ksonnet](https://github.com/ksonnet/ksonnet) applications (deprecated)
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

