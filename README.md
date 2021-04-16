# helm-dependency-submodules

Demonstrates the usage of Git submodules as Helm dependencies

## Overview

Helm has a concept of [dependencies](https://helm.sh/docs/helm/helm_dependency) to manage the deployment of multiple charts. While this is typically accomplished by referencing a subchart from a helm repository, subcharts can also be referenced from the file system. Instead of copying the contents directly, a reference to an existing Git repository can be made using [Git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules).

The Helm chart contained within this repository will deploy a simple Quarkus based application that is contained within a submodule consisting of a set of [Helm charts from the Red Hat Developers team](https://github.com/redhat-developer/redhat-helm-charts).

## Installation and Deployment

The following sections describe the various way in which the chart can be deployed:

### Standalone

First clone the repository for your machine and change into the repository directory:

```shell
git clone --recurse-submodules https://github.com/sabre1041/helm-dependency-submodules
cd helm-dependency-submodules
```

_Note:_ The use of the the `--recurse-submodules` flag will automatically initialize the submodule and update the content to the associated commit.

Update the Helm dependencies from the chart located in the submodule

```shell
helm dependency update
```

Create a new Helm release in a new namespace called 'helm-dependency-submodules-standalone`:

```shell
helm upgrade -i --create-namespace -n helm-dependency-submodules-standalone quarkus .
```

Confirm the application is running

```shell
kubectl get pods -n helm-dependency-submodules-standalone
```

Navigate to the application

```shell
kubectl get routes -n helm-dependency-submodules-standalone -o jsonpath='https://{ .items[*].spec.host }'
```


### ArgoCD

The deployment of the chart also be facilitated using a GitOps tool, such as [ArgoCD](https://argoproj.github.io/argo-cd/).

**Important**: ArgoCD tracks applications using the `app.kubernetes.io/instance` annotation on resources. If both the standalone method as described above and ArgoCD facilitates the deployment with the same name on the same cluster (even in different namespaces [in cluster level mode]), the ArgoCD application will appear to be out of sync. To mitigate this issue, set the `application.instanceLabelKey` in the `argocd-cm` ConfigMap or the `spec.applicationInstanceLabelKey:` when using the ArgoCD operator to the value of `argocd.argoproj.io/instance`.

Deploy the instance of ArgoCD with access to create resources across the cluster.

Apply the Application resource which will deploy the Chart to a namespace called `helm-dependency-submodules-argocd`.

```shell
kubectl apply -n <argocd_namespace> -f argocd/application.yaml
```

Confirm the application has been deployed and is running

```shell
kubectl get pods -n helm-dependency-submodules-argocd
```

Navigate to the application

```shell
kubectl get routes -n helm-dependency-submodules-argocd -o jsonpath='https://{ .items[*].spec.host }'
```


