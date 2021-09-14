---
title: "8. Tracking and Deployment Strategies"
weight: 8
sectionnumber: 8
---

If you are using ArgoCD with Git or Helm tools, ArgoCD gives you the availability to configure different tracking and deployment strategies.


## Task {{% param sectionnumber %}}.1: Helm

For Helm you can use Semver versions either to pin to a specific version or to track minor / patch changes.

| Use Case                                   | How                    | Examples                |
|--------------------------------------------|------------------------|-------------------------|
| Pin to a version (e.g. in production)      | Use the version number | 1.2.0                   |
| Track patches (e.g. in pre-production)     | Use a range            | 1.2.* or >=1.2.0 <1.3.0 |
| Track minor releases (e.g. in QA)          | Use a range            | 1.* or >=1.0.0 <2.0.0   |
| Use the latest (e.g. in local development) | Use star range         | * or >=0.0.0            |


## Task {{% param sectionnumber %}}.2: Git


| Use Case                                   | How                                                                                     | Notes                      |
|--------------------------------------------|-----------------------------------------------------------------------------------------|----------------------------|
| Pin to a version (e.g. in production)      | Either (a) tag the commit with (e.g. v1.2.0) and use that tag, or (b) using commit SHA. | See commit pinning.        |
| Track patches (e.g. in pre-production)     | Tag/re-tag the commit, e.g. (e.g. v1.2) and use that tag.                               | See tag tracking           |
| Track minor releases (e.g. in QA)          | Re-tag the commit as (e.g. v1) and use that tag.                                        | See tag tracking           |
| Use the latest (e.g. in local development) | Use HEAD or master (assuming master is your master branch).                             | See HEAD / Branch Tracking |


### Head / Branch Tracking

Either a branch name or a symbolic reference (like HEAD). For branches ArgoCD will take the latest commit of this branch.
This method is often used in development environment where you want to apply the latest changes.


### Tag Tracking

The state at the specified Git tag will be applied to the cluster. This provides some advantages over branch tracking. Tags are less frequently updated and more stable than a tracked branch. It is more suitable for staging environments due the capability of tracking minor and patch releases.


### Commit Pinning

Commit or version pinning can achieved in two ways. Eiteher you van specifiy the full semver Git tag (v1.2.0) or a commit SHA. Usually the Git tag offers more flexibility while the commit SHA offers more immutuability. Commit pinning is generally the first choice for production environments.


## Task {{% param sectionnumber %}}.3: Helm
