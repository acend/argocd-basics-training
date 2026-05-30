---
title: "Setup"
weight: 1
type: docs
menu:
  main:
    weight: 1
---

When not using the preinstalled Web IDE provided by your trainer, it's also possible to use your own computer. If you're using the provided setup, you can skip this section.


## Local Environment Requirements

In this training it is required to have the following tools installed on your computer:

* git
* git bash on Windows
{{% onlyWhenNot no-argocd-cli %}}
* Argo CD CLI
{{% /onlyWhenNot %}}
{{% onlyWhen openshift %}}
* oc Tool (OpenShift Client)
{{% /onlyWhen %}}
* kubectl

{{% onlyWhenNot no-argocd-cli %}}


## Argo CD Command line tool

Follow the instructions on [this](https://argo-cd.readthedocs.io/en/stable/cli_installation/) page to install the ArgoCD tool on your local computer.
{{% /onlyWhenNot  %}}

{{% onlyWhen openshift %}}


## oc tool

Follow the instructions on [this](https://openshift-basics.training.acend.ch/setup/) page to install the oc tool on your local computer.
{{% /onlyWhen  %}}


## kubectl

Follow the instructions on [this](https://kubernetes-basics.training.acend.ch/setup/) page to install the kubectl on your local computer.

