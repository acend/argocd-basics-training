---
title: "Getting started"
weight: 1
onlyWhen: centris-init
---


## {{% task %}} Setting up the local IDE

The first thing we're going to do is to explore our lab environment and get in touch with the different components.

The namespace with the name corresponding to your username is going to be used for all the hands-on labs. And you will be using the ArgoCD webconsole, to verify what resources and objects Argo CD created for you.

{{% alert title="Note" color="info" %}}Make sure you completed [the setup](../../setup/) before you continue with this lab.{{% /alert %}}


Once you've successfully set up the environment in your IDE, open a new terminal and check the installed {{% param cliToolName %}}version by executing the following command:

```bash
{{% param cliToolName %}} version
```

Ensure you have all of the following tools:

* oc
* kubectl
* kustomize
* helm


### Task 1.1.1: Local Workspace Directory

During the lab, you’ll be using local files (eg. YAML resources) which will be applied in your lab project.

Create a new folder on your local machine  (for example `argocd-training`).

```bash
mkdir argocd-training && cd argocd-training
```


### Task 1.1.2: Login to ArgoCD

{{% onlyWhen no-argocd-cli %}}
You can access Argo CD via the Web UI. Open your browser and navigate to [https://{{% param argoCdUrl %}}](https://{{% param argoCdUrl %}}) and login with the credentials provided by your trainer.
{{% /onlyWhen %}}


### Task 1.1.3: Lab Setup


Most of the labs will be done inside the {{% param distroName %}} project with your username. Verify that your oc tool is configured to point to the right project:


```s
oc project
```


```
Using project "<username>" on server "https://<theClusterAPIURL>".
```

The returned project name should correspond to your username.

