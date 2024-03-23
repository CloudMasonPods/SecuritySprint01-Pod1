<!--- app-name: Harbor -->

# Bitnami package for Harbor

Harbor is an open source trusted cloud-native registry to store, sign, and scan content. It adds functionalities like security, identity, and management to the open source Docker distribution.

[Overview of Harbor](https://goharbor.io/)

## TL;DR

```console
helm install my-release oci://registry-1.docker.io/bitnamicharts/harbor
```

Looking to use Harbor in production? Try [VMware Tanzu Application Catalog](https://bitnami.com/enterprise), the enterprise edition of Bitnami Application Catalog.

## Introduction

This [Helm](https://github.com/kubernetes/helm) chart installs [Harbor](https://github.com/goharbor/harbor) in a Kubernetes cluster. Welcome to [contribute](https://github.com/bitnami/charts/blob/main/CONTRIBUTING.md) to Helm Chart for Harbor.

This Helm chart has been developed based on [goharbor/harbor-helm](https://github.com/goharbor/harbor-helm) chart but including some features common to the Bitnami chart library.
For example, the following changes have been introduced:

- Possibility to pull all the required images from a private registry through the  Global Docker image parameters.
- Redis&reg; and PostgreSQL are managed as chart dependencies.
- Liveness and Readiness probes for all deployments are exposed to the values.yaml.
- Uses new Helm chart labels formatting.
- Uses Bitnami container images:
  - non-root by default
  - published for debian-10 and ol-7
- This chart support the Harbor optional components.

Bitnami charts can be used with [Kubeapps](https://kubeapps.dev/) for deployment and management of Helm Charts in clusters.

## Prerequisites

- Kubernetes 1.23+
- Helm 3.8.0+
- PV provisioner support in the underlying infrastructure
- ReadWriteMany volumes for deployment scaling

## Installing the Chart

To install the chart with the release name `my-release`:

```console
helm install my-release oci://REGISTRY_NAME/REPOSITORY_NAME/harbor
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
helm delete --purge my-release
```

Additionally, if `persistence.resourcePolicy` is set to `keep`, you should manually delete the PVCs.


```console
helm install my-release \
  --set adminPassword=password \
    oci://REGISTRY_NAME/REPOSITORY_NAME/harbor
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

The above command sets the Harbor administrator account password to `password`.

> NOTE: Once this chart is deployed, it is not possible to change the application's access credentials, such as usernames or passwords, using Helm. To change these application credentials after deployment, delete any persistent volumes (PVs) used by the chart and re-deploy it, or use the application's built-in administrative tools if available.

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart. For example,

```console
helm install my-release -f values.yaml oci://REGISTRY_NAME/REPOSITORY_NAME/harbor
```
> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use 