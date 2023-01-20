---
title: "5.3 Jsonnet (Optional)"
weight: 53
sectionnumber: 5.3
---

This lab explains how to use [jsonnet](https://jsonnet.org/)  as manifest format together with Argo CD.


## Jsonnet

[Jsonnet](https://jsonnet.org/) is a templating language which adds the possibility to programmatically work with the underlying data. It basically is a simple extension of [JSON](https://json.org).

Let's have a look at an example first. The following jsonnet file

```json
{
  application1: {
    name: "jsonnet-application1",
    basepath: "/application/test",
    path: self.basepath + "/" + self.name,
  },
  application2: self.application1 { name: "jsonnet-application2" },
}
```

will render into:

```json
{
  "application1": {
    "basepath": "/application/test",
    "name": "jsonnet-application1",
    "path": "/application/test/jsonnet-application1"
  },
  "application2": {
    "basepath": "/application/test",
    "name": "jsonnet-application2",
    "path": "/application/test/jsonnet-application2"
  }
}
```

Among many other features, Jsonnet can help to reduce duplications.


### Further Docs

Read more about the jsonnet integration in the [official documentation](https://argoproj.github.io/argo-cd/user-guide/jsonnet/)


## Task {{% param sectionnumber %}}.1: Deploy the simple-example with jsonnet

Let's first explore the files in your local repository under `jsonnet`.

Similar to `helm`, `jsonnet` allows us to extract parameters into a separate file. In our example we extracted all values to the `params.libsonnet` file:

```json
{
  containerPort: 5000,
  image: "quay.io/acend/example-web-go",
  name: "argo-jsonnet-example-<username>",
  replicas: 1,
  servicePort: 5000,
  type: "ClusterIP",
}
```

And the actual template file, containing the kubernetes service and deployment definitions, `simple-application.jsonnet` file:

```json
local params = import 'params.libsonnet';

[
   {
      "apiVersion": "v1",
      "kind": "Service",
      "metadata": {
         "name": params.name
      },
      "spec": {
         "ports": [
            {
               "port": params.servicePort,
                "protocol": "TCP",
               "targetPort": params.containerPort
            }
         ],
         "selector": {
            "app": params.name
         },
         "type": params.type
      }
   },
   {
      "apiVersion": "apps/v1",
      "kind": "Deployment",
      "metadata": {
         "name": params.name
      },
      "spec": {
         "replicas": params.replicas,
         "revisionHistoryLimit": 3,
         "selector": {
            "matchLabels": {
               "app": params.name
            },
         },
         "template": {
            "metadata": {
               "labels": {
                  "app": params.name
               }
            },
            "spec": {
               "containers": [
                  {
                     "image": params.image,
                     "name": params.name,
                     "ports": [
                     {
                        "containerPort": params.containerPort
                     }
                     ]
                  }
               ]
            }
         }
      }
   }
]
```

Note the first line, where we tell jsonnet where to get params from.

Ok now replace the `<username>` placeholder in the `params.libsonnet` file with your username.

Commit and push the changes to your repository.

{{% details title="Hint" %}}
```bash
git add helm/simple-example/values.yaml
git commit -m "Change Username"
git push
```
{{% /details %}}


Create the new Argo CD application.

```bash
argocd app create argo-jsonnet-$STUDENT --repo https://{{% param giteaUrl %}}/$STUDENT/argocd-training-examples.git --path 'jsonnet' --dest-server https://kubernetes.default.svc --dest-namespace $STUDENT
```

Sync the application

{{% details title="Hint" %}}

To sync (deploy) the resources you can simply click sync in the web UI or execute the following command:

```bash
argocd app sync argo-jsonnet-$STUDENT
```
{{% /details %}}

And verify whether your jsonnet Application definition has be successfully synced.


## Task {{% param sectionnumber %}}.2: Autosync and scale up

Tell the application to sync automatically, to enable self-healing and auto-prune

{{% details title="Hint" %}}
```bash
argocd app set argo-jsonnet-$STUDENT --sync-policy automated
argocd app set argo-jsonnet-$STUDENT --self-heal
argocd app set argo-jsonnet-$STUDENT --auto-prune
```
{{% /details %}}

Now let's change the replicacount of the deployment and scale to `2` pods.

Change the replica param in your `params.libsonnet` to 2

```json
{
  containerPort: 5000,
  image: "quay.io/acend/example-web-go",
  name: "argo-jsonnet-example-<username>",
  replicas: 2,
  servicePort: 5000,
  type: "ClusterIP",
}
```

Commit and push the changes to your repository.

{{% details title="Hint" %}}
```bash
git add helm/simple-example/values.yaml
git commit -m "Scale Jsonnet to two"
git push
```
{{% /details %}}

And verify the result in the ArgoCD Ui or by using the following command, this might take a little while to happen, depending on how many trainees are currently working on the labs. Hint: hit refresh to speed up the process.

```bash
{{% param cliToolName %}} get pod --namespace $STUDENT --watch
```


## Task {{% param sectionnumber %}}.4: Delete the Applications

Delete the applications after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
```bash
argocd app delete argo-jsonnet-$STUDENT
```
{{% /details %}}
