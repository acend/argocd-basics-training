---
title: "3. Complex Example"
weight: 3
sectionnumber: 3
---

In this lab we're going to deploy and manage a complex  application with Argo CD.


## Complex Example

The complex example consists of the following components:

* data-consumer: deployment, configmap, service and route
* data-producer: deployment, service and route
* kafka-cluster: Kafka CRD, Kafka topic
* jaeger: Jaeger Tracing CRD

In the same git repository you cloned/forked/mirrored in the previous labs, you also find the manifests for the more complex application under `complex-application`.


## Task {{% param sectionnumber %}}.1: Create Argo CD application for the complex example

First you'll have to create a new Argo CD application.

{{% alert  color="primary" title="Note" %}}
We're going to deploy the complex example into the **same namespace** as we did with the simple example.
Make sure the variable `LAB_USER` is set correctly
```bash
export LAB_USER=<username>
echo $LAB_USER
```
or replace $LAB_USER in every command accordingly
{{% /alert %}}


```bash
argocd app create argo-complex-$LAB_USER --repo https://{{% param giteaUrl %}}/$LAB_USER/argocd-training-examples.git --path 'complex-application' --dest-server https://kubernetes.default.svc --dest-namespace $LAB_USER
```

Expected output: `application 'argo-complex-<username>' created`

Explore the Argo application and its resources in the web UI or via CLI.

{{% details title="Hint" %}}

```bash
argocd app get argo-complex-$LAB_USER
```
{{% /details %}}


## Task {{% param sectionnumber %}}.2: Sync the application

The application status is in OutOfSync state. Let's sync it, and make sure the resources are deployed on the cluster.

{{% details title="Hint" %}}

To sync (deploy) the resources you can simply click sync in the web UI or execute the following command:

```bash
argocd app sync argo-complex-$LAB_USER
```
{{% /details %}}

Let's also make sure that the application will be **synced** automatically, by defining a so called `sync-policy`.

{{% details title="Hint" %}}

The `sync-policy` can be either set in the web UI or via CLI:

```bash
argocd app set argo-complex-$LAB_USER --sync-policy automated
```
{{% /details %}}

Also make sure `self-healing` and `auto-pruning` is enabled.

{{% details title="Hint" %}}


```bash
argocd app set argo-complex-$LAB_USER --self-heal
argocd app set argo-complex-$LAB_USER --auto-prune
```
{{% /details %}}


## Task {{% param sectionnumber %}}.3: Scale up the data-producer deployment

Let's now test the setup and scale the data-producer deployment to `2` replicas by editing the `argocd-training-examples/complex-application/producer.yaml`

{{% details title="Hint" %}}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: data-producer
    application: amm-techlab
  name: data-producer
spec:
  replicas: 2
  ...
```
{{% /details %}}

Commit and push the changes to your repository.

{{% details title="Hint" %}}
```bash
git add complex-application/producer.yaml
git commit -m "Increased replicas to 2"
git push
```
{{% /details %}}


## Task {{% param sectionnumber %}}.4: Delete the Application

Delete the application after you've explored the Argo CD Resources and the managed Kubernetes resources.

{{% details title="Hint" %}}
```bash
argocd app delete argo-complex-$LAB_USER
```
{{% /details %}}
