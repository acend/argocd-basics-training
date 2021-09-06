---
title: "1. Getting started"
weight: 1
sectionnumber: 1
---


## Task {{% param sectionnumber %}}.1: Login

The first thing we're going to do is to explore our lab environment and get in touch with the different components.

Login to the web console of the Lab Cluster with the provided Username and Password:

<!-- markdownlint-disable MD034 -->
[https://{{% param techlabClusterWebConsolePrefix %}}.{{% param techlabClusterDomainName %}}](https://{{% param techlabClusterWebConsolePrefix %}}.{{% param techlabClusterDomainName %}})
<!-- markdownlint-enable MD034 -->

{{% alert title="Note" color="primary" %}} Ask your trainer if you don't have your Username and Password {{% /alert %}}

The project with the name corresponding to your username is going to be used for all the hands-on labs. And you will be using the `oc tool` or the OpenShift webconsole, to verify what resourcen and objects Argo CD created for you.


### Task {{% param sectionnumber %}}.1.1: Web IDE

{{% alert title="Note" color="primary" %}}You can also use your local installation of the cli tools. Make sure you completed [the setup](../../setup/) before you continue with this lab.{{% /alert %}}

As your lab environment, we use a so-called web IDE, directly deployed on the lab environment. To login to your specific web IDE, we need to figure out the IDE Password, which is configured as Environment Variable in the Deployment `amm-techlab-ide` in your project.

Go and get the value out of the Environment Variable and log into the Web IDE.

{{% alert title="Note" color="primary" %}}Use Chrome for the best experience. The Url to the Web IDE also can be found in your project. The deployment is exposed with a route. {{% /alert %}}


Once you're successfully logged into the web IDE open a new Terminal by hitting `CTRL + SHIFT + C` or clicking the Menu button --> Terminal --> new Terminal and check the installed oc version by executing the following command:

```bash
oc version
```

The Web IDE Pod consists of the following tools:

* oc
* kubectl
* kustomize
* helm
* kubectx
* kubens
* tekton cli
* odo
* argocd

The files in the home directory under `/home/coder` are stored in a persistence volume.


### Task {{% param sectionnumber %}}.1.2: Login on OpenShift with oc tool

The easiest way to login to the lab cluster using the oc tool is, by copying the login command from the web console (Click on the Username in the top right corner of your web console and then Copy Login Command, to get to the login command).

Paste this login command in the Terminal and verify the output `Logged into ...`

> If you are using Firefox, you can paste the command with `shift + insert`

Switch to your project with `oc project <username>`

If you want to use your local `oc` tool, make sure to get the appropriate version.


### Task {{% param sectionnumber %}}.1.3: Local Workspace Directory

During the lab, you’ll be using local files (eg. YAML resources) which will be applied in your lab project.

Create a new folder for your \<workspace> in your Web IDE  (for example ./amm-techlab/). Either you can create it with `right-mouse-click -> New Folder` or in the Web IDE terminal `mkdir argocd-techlab`


### Task {{% param sectionnumber %}}.1.4: Login on ArgoCD using argocd CLI

You can access Argo CD via Web UI (URL is provided by your teacher) or using the CLI. The Argo CD CLI Tool is already installed on the web IDE.

Since the sso login does not work inside the Web IDE for various reasons, your teacher will provide a generic local Argo CD account `hannelore` without any number.

```bash
argocd login {{% param techlabArgoCdUrl %}} --grpc-web --username hannelore
```

{{% alert title="Note" color="primary" %}}Make sure to pass the `<ARGOCD_SERVER>` without protocol e.g. `argocd.domain.com`. The `--grpc-web` parameter is necessary due to missing http 2.0 router.{{% /alert %}}


#### Task {{% param sectionnumber %}}.1.4.1: Login on your local computer (Optional)

{{% alert title="Warning" color="secondary" %}}
Please make sure you completed [the setup](../../setup/) before you continue with this lab.
{{% /alert %}}

You can access Argo CD via Web UI (URL is provided by your teacher) or using the CLI. For CLI usage use the following command to login (credentials are given by your teacher):

```bash
argocd login {{% param techlabArgoCdUrl %}} --sso --grpc-web
```

{{% alert title="Note" color="primary" %}}Make sure to pass the `<ARGOCD_SERVER>` without protocol e.g. `argocd.domain.com`. Follow the sso login steps in the new browser window. The `--grpc-web` parameter is necessary due to missing http 2.0 router.{{% /alert %}}


### Task {{% param sectionnumber %}}.1.5: Lab Setup

Most of the labs will be done inside the OpenShift project with your username. Verify that your oc tool is configured to point to the right project:

```s
oc project
```

```
Using project "<username>" on server "https://<theClusterAPIURL>".
```

The returned project name should correspond to your username.


## Task {{% param sectionnumber %}}.2: Argo CD CLI

The [Argo CD CLI](https://argoproj.github.io/argo-cd/cli_installation/) is a powerful tool to manage Argo CD and different applications. It's a self contained binary written in Go and available for Linux, Mac OS and Windows. Thanks to the fact that the CLI is implemented in Go, it can be easily integrated into scripts and build servers for automation purposes.


### Task {{% param sectionnumber %}}.2: Getting familiar with the CLI

Print out the help of the CLI by typing

```bash
argocd --help
```

You will see a list with the available commands and flags. If you prefer to browse the manual in the browser you'll find it in the [online documentation](https://argoproj.github.io/argo-cd/user-guide/commands/argocd/).


The `--help` flag is available for every command and subcommand of the CLI. Beside the documentation for every flag and subcommand in the current context, it prints out example command lines for the most common use cases.

Now get the help of the `app create` subcommand and find the examples and documentation of the flags.

```bash
argocd app create --help
```


### Task {{% param sectionnumber %}}.2: Autocomplete

A productivity booster when working with the CLI is the autocompletion feature. It can be used for `bash` and `zsh` shells. You can enable the autocompletion for the current `bash` with the following command:

```bash
source <(argocd completion bash)
```

After typing `argocd` you can autocomplete the subcommands with a double tap the tabulator key. This works even for deployed artifacts on the cluster: A double tab after `argocd app get` lists all defined Argo CD applications.

To install autocompletion permanently you can add the following command in the `~/.bashrc` file.

```bash
echo "source <(argocd completion bash)" >> ~/.bashrc
source ~/.bashrc
```

Find further information in the [official documentation](https://argoproj.github.io/argo-cd/user-guide/commands/argocd_completion/)
