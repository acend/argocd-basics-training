---
title: "Getting started"
weight: 1
onlyWhen: getting-started
---

## Lab Components

Our lab setup consists of the following components:


* Git Server ([Gitea](https://gitea.io)): [https://{{% param giteaUrl %}}](https://{{% param giteaUrl %}}/) -> use "sign up with dex" to sign in
* Argo CD Server: [https://{{% param argoCdUrl %}}](https://{{% param argoCdUrl %}}) -> use "login with OpenShift" to sign in
{{% onlyWhen openshift %}}
* OpenShift Cluster: [https://{{% param ocpConsoleUrl %}}](https://{{% param ocpConsoleUrl %}})
{{% /onlyWhen %}}
* The WebIDE Development Environment with all necessary tools installed will be provided by the trainer -> use "login with OpenShift" to sign in


## {{% task %}} Web IDE

{{% onlyWhen openshift %}}

The very first thing you need to do is to log in to the OpenShift Console: [https://{{% param ocpConsoleUrl %}}](https://{{% param ocpConsoleUrl %}})

This is necessary to ensure your account is created and can be used by the other tools to authenticate.

{{% /onlyWhen %}}

The first thing we're going to do now is to explore our lab environment and get in touch with the different components.

The namespace with the name corresponding to your username is going to be used for all the hands-on labs. And you will be using {{% onlyWhenNot no-argocd-cli %}} the `argocd tool` or {{% /onlyWhenNot %}} the ArgoCD webconsole, to verify what resources and objects Argo CD created for you.

{{% alert title="Note" color="info" %}}You can also use your local installation of the cli tools. Make sure you completed [the setup](../../setup/) before you continue with this lab.{{% /alert %}}

{{% alert title="Note" color="info" %}}The URL and Credentials to the Web IDE will provided by the teacher. Use Chrome for the best experience.{{% /alert %}}


Once you're successfully logged into the web IDE open a new Terminal by hitting `CTRL + SHIFT + ¨` or clicking the Menu button --> Terminal --> new Terminal and check the installed {{% param cliToolName %}}version by executing the following command:

```bash
{{% param cliToolName %}} version
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

The files in your root directory (`/home/project`) are stored on a persistent volume, so all your data in this directory will be persistent for when you open the webshell again.


### Task 1.1.1: Local Workspace Directory

During the lab, you’ll be using local files (eg. YAML resources) which will be applied in your lab project.

Create a new folder for your `<workspace>` in your Web IDE. Either you can create it with `right-mouse-click -> New Folder` or in the Web IDE terminal.

```bash
mkdir argocd-training && cd argocd-training
```


### Task 1.1.2: Login to ArgoCD

{{% onlyWhenNot no-argocd-cli %}}
You can access Argo CD via Web UI (Credentials are provided by your teacher) or using the CLI. The Argo CD CLI Tool is already installed on the web IDE.

```bash
argocd login {{% param argoCdUrl %}} --grpc-web --username $USER
```
{{% /onlyWhenNot %}}

{{% onlyWhen no-argocd-cli %}}
You can access Argo CD via the Web UI. Open your browser and navigate to [https://{{% param argoCdUrl %}}](https://{{% param argoCdUrl %}}) and login with the credentials provided by your trainer.
{{% /onlyWhen %}}
{{% onlyWhen openshift %}}


### Task 1.1.3: Lab Setup


Most of the labs will be done in your personal {{% param distroName %}} project. Your WebShell is preconfigured and logged into the lab {{% param distroName %}} cluster.

Verify that your oc tool is configured to point to the right project:


```s
oc project
```


```
Using project "<username>" on server "https://kubernetes.default".
```

The returned project name should correspond to your username equal to the env variable `echo $USER`.
{{% /onlyWhen  %}}


{{% onlyWhenNot no-argocd-cli %}}


## {{% task %}} Argo CD CLI

The [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/) is a powerful tool to manage Argo CD and different applications. It's a self contained binary written in Go and available for Linux, Mac OS and Windows. Thanks to the fact that the CLI is implemented in Go, it can be easily integrated into scripts and build servers for automation purposes.


### Task 1.2.1: Getting familiar with the CLI

Print out the help of the CLI by typing

```bash
argocd --help
```

You will see a list with the available commands and flags. If you prefer to browse the manual in the browser you'll find it in the [online documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/).

```
argocd controls a Argo CD server

Usage:
  argocd [flags]
  argocd [command]

Available Commands:
  account     Manage account settings
  admin       Contains a set of commands useful for Argo CD administrators and requires direct Kubernetes access
  app         Manage applications
  cert        Manage repository certificates and SSH known hosts entries
  cluster     Manage cluster credentials
  completion  output shell completion code for the specified shell (bash or zsh)
  context     Switch between contexts
  gpg         Manage GPG keys used for signature verification
  help        Help about any command
  login       Log in to Argo CD
  logout      Log out from Argo CD
  proj        Manage projects
  relogin     Refresh an expired authenticate token
  repo        Manage repository connection parameters
  repocreds   Manage repository connection parameters
  version     Print version information

Flags:
      --auth-token string               Authentication token
      --client-crt string               Client certificate file
      --client-crt-key string           Client certificate key file
      --config string                   Path to Argo CD config (default "/home/bbuehlmann/.argocd/config")
      --core                            If set to true then CLI talks directly to Kubernetes instead of talking to Argo CD API server
      --grpc-web                        Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2.
      --grpc-web-root-path string       Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2. Set web root.
  -H, --header strings                  Sets additional header to all requests made by Argo CD CLI. (Can be repeated multiple times to add multiple headers, also supports comma separated headers)
  -h, --help                            help for argocd
      --http-retry-max int              Maximum number of retries to establish http connection to Argo CD server
      --insecure                        Skip server certificate and domain verification
      --logformat string                Set the logging format. One of: text|json (default "text")
      --loglevel string                 Set the logging level. One of: debug|info|warn|error (default "info")
      --plaintext                       Disable TLS
      --port-forward                    Connect to a random argocd-server port using port forwarding
      --port-forward-namespace string   Namespace name which should be used for port forwarding
      --server string                   Argo CD server address
      --server-crt string               Server certificate file
```

The `--help` flag is available for every command and subcommand of the CLI. Beside the documentation for every flag and subcommand in the current context, it prints out example command lines for the most common use cases.

Now get the help of the `app create` subcommand and find the examples and documentation of the flags.

```bash
argocd app create --help
```


### Task 1.2.2: Autocompletion

{{% alert title="Note" color="info" %}}This step is only needed, when you're not working with the Web IDE we've provided. The autocompletion is already installed in the Web IDE{{% /alert %}}

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

Find further information in the [official documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd_completion/)

{{% /onlyWhenNot %}}
