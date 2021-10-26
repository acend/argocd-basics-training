---
title: "8. Tracking and Deployment Strategies"
weight: 8
sectionnumber: 8
---

If you are using ArgoCD with Git or Helm tools, ArgoCD gives you the availability to configure different tracking and deployment strategies.


## Helm

For Helm you can use Semver versions either to pin to a specific version or to track minor / patch changes.

| Use Case                                   | How                    | Examples                |
|--------------------------------------------|------------------------|-------------------------|
| Pin to a version (e.g. in production)      | Use the version number | 1.2.0                   |
| Track patches (e.g. in pre-production)     | Use a range            | 1.2.* or >=1.2.0 <1.3.0 |
| Track minor releases (e.g. in QA)          | Use a range            | 1.* or >=1.0.0 <2.0.0   |
| Use the latest (e.g. in local development) | Use star range         | * or >=0.0.0            |


## Git


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


## Task {{% param sectionnumber %}}.1: Git tracking

In this task we're going to configure a version tracking with a Git tag. The goal of this task to show you how to tack the patch version from a Git tag and therefore freeze the deployment to specific commits.

First we create a Git tag `v1.0.0` and push the tag to the repository.

{{% details title="Hint" %}}

To create and push a Git tag execute the following command:
```bash
git tag v1.0.0
git push origin --tags
```
{{% /details %}}

You will receive an error when trying to create the new application
```
FATA[0000] rpc error: code = InvalidArgument desc = application spec is invalid: InvalidSpecError: application repo https://github.com/acend/argocd-training-examples.git is not permitted in project 'project-hannelore15';InvalidSpecError: application destination {https://kubernetes.default.svc hannelore15} is not permitted in project 'project-hannelore15'
```

To track the v1.0 patch version tag on our application execute the following command:

```bash
argocd app set argo-complex-$LAB_USER --revision v1.0
```
{{% /details %}}


Increase the number of replicas in your file `<workspace>/complex-application/producer.yaml` to 2.
After that commit and push your changes to the Git repository.

{{% details title="Hint" %}}

{{< highlight YAML "hl_lines=9" >}}
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: data-producer
    application: amm-techlab
  name: data-producer
spec:
  replicas: 2
  selector:
    matchLabels:
      deployment: data-producer
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        application: amm-techlab
        deployment: data-producer
        app: data-producer
    spec:
      containers:
        - image: quay.io/puzzle/quarkus-techlab-data-producer:jaegerkafka
          imagePullPolicy: Always
          env:
            - name: PRODUCER_JAEGER_ENABLED
              value: 'true'
          livenessProbe:
            failureThreshold: 5
            httpGet:
              path: /health/live
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 3
            periodSeconds: 20
            timeoutSeconds: 15
          readinessProbe:
            failureThreshold: 5
            httpGet:
              path: /health/ready
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 3
            periodSeconds: 20
            timeoutSeconds: 15
          name: data-producer
          ports:
            - containerPort: 8080
              name: http
              protocol: TCP
          resources:
            limits:
              cpu: '1'
              memory: 500Mi
            requests:
              cpu: 50m
              memory: 100Mi
{{< / highlight >}}

For commiting and pushing your changes to your Git repository, execute follwing command:

```bash
git add . && git commit -m "scale producer replicas to 2" && git push origin
```

{{% /details %}}

Now you can try to sync your applicaion with following command:

```bash
argocd app sync argo-complex-$LAB_USER
```

Check the number of configured replicas on the consumer deployment.

{{% details title="Hint" %}}
To see the number of configured replicas execute follwing command:

```bash
kubectl describe deployment producer
```

You can see in the command output, the number of replicas didn't changed and remains to one. This is because we only track tagged version `1.0.*` or `>=1.0.0 <1.1.0.`

{{< highlight YAML "hl_lines=1" >}}
TODO: insert command output
{{< / highlight >}}


{{% /details %}}

Let ArgoCD pickup the latest change, for that we have to create a git tag that is tracked with the configured tracking stratiegie.
Let's create a new Git tag with the patch version `v.1.0.1` and push it to the repsitory. Then resync the ArgoCD app and checkt the status.

{{% details title="Hint" %}}
Execute the following command to create and push a new Git tag

```bash
git tag v1.0.1 && git push origin --tags
```

{{% /details %}}

With the new created tag, ArgoCD is goingt to pick up and apply the latest changes and scales up the replica count to 2.
First let us sync the changes and check if the ArgoCD App is in Sync.

```bash
argocd app sync argo-complex-$LAB_USER
```

Then diplay the status with following command:

```bash
argocd app get argo-complex-$LAB_USER
```

If the app is in sync, you can check the number of replicas of the producer deployment.


```bash
kubectl describe deployment producer
```

Now you can see in the output that the replica count has changed to 2.

```bash
TODO
```


```
...
GROUP  KIND        NAMESPACE    NAME            STATUS   HEALTH   HOOK  MESSAGE
       Service     hannelore15  simple-example  Unknown  Missing        Resource :Service is not permitted in project project-hannelore15.
apps   Deployment  hannelore15  simple-example  Synced   Healthy
FATA[0001] Operation has completed with phase: Failed
```


## Task {{% param sectionnumber %}}.2: Cleanup


Let us clean up the tracking task.

First remove the tracked version and set the revision to the HEAD reference. So ArgoCD is tracking again the latest commit of the configured branch. And then set the producer replica count back to 1 and commit your changes.

{{% details title="Hint" %}}
To remove the pinned version on our application execute the following command:

```bash
argocd app set argo-complex-$LAB_USER --revision HEAD
```

Open yout producer.yaml file and set the replica count back to 1.
```yaml
Deployment.yaml
```

At last commit and push your changes with the following command:
```bash
git add . && git commit -m "revert replica count to 1" && git push origin
```

{{% /details %}}
