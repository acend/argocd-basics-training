---
title: "13. CLI"
weight: 13
sectionnumber: 13
onlyWhen: implemented
---

## Argo CD CLI

This lab contains some important cli commands of the `argocd` binary.

### Autocompletion

Autocompletion is available for `bash` and `zsh` shell. Enable autocompletion for `bash`:

```bash
source <(argocd completion bash)
```

After typing `argocd` you can autocomplete the subcommands with a double tap the tabulator key. This works even for deployed artifacts on the cluster: A double tab after `argocd app get` lists all defined Argo CD applications.

### Manage Applications

List all applications
```bash
argocd app list
```

Sync the manifests of a selected application
```bash
argocd app sync pitc-argocd-lab-test
```

Edit App manifest
```bash
argocd app edit pitc-argocd-lab-test
```

Follow log output of an application
```bash
argocd app logs pitc-argocd-lab-test --follow
```

Diff the live manifest with the desired manifest in a diff tool. In this case we use Visual Studio Code.

```bash
export KUBECTL_EXTERNAL_DIFF="/snap/bin/code -d"
argocd app diff pitc-argocd-lab-test
```

Show the Manifests of an application
```bash
argocd app manifests pitc-argocd-lab-test
```

Show all resources of an application

```bash
argocd app resources pitc-argocd-lab-test
```

```
GROUP  KIND        NAMESPACE             NAME            ORPHANED
       Service     pitc-argocd-lab-test  simple-example  No
apps   Deployment  pitc-argocd-lab-test  simple-example  No
```
