---
title: "Kubernetes Deployment"
weight: 300
summary: "Overview of OpenRun deployment on Kubernetes"
---

## Installation

OpenRun can be installed on a Kubernetes cluster using the Helm chart. Running the Helm chart creates:

- An service which run the OpenRun API server
- Optionally, a Postgres database for metadata. An external Postgres database can be used instead.
- Optionally, a container registry is installed. An external registry can be used instead.

SQLite based metadata is not supported when using Kubernetes. The Postgres metadata needs to be managed properly, with backups and scheduled upgrades. An externally managed Postgres installation is recommended for production Kubernetes OpenRun installations.

A container registry is required for Kubernetes based OpenRun install. The registry can be installed through the Helm chart or an external registry can be used. An installation with external Postgres and Registry is shown below.

<picture  class="responsive-picture" style="display: block; margin-left: auto; margin-right: auto;">
  <img alt="Kubernetes Deployment" src="/d2/k8s.svg">
</picture>

## Configuration

```toml {filename="openrun.toml"}
[system]
container_command = "kubernetes"
```

is the main config which enables Kubernetes mode. By default. Kaniko based builds are used. Delegated builds can be configured instead, see [delegated builds]({{< ref "/docs/container/build/#delegated-build-mode" >}}). See [registry]({{< ref "/docs/container/build/#config" >}}) for registry config.

## Architecture

OpenRun is installed as a Kubernetes Deployment, with a Service for routing API calls. For each containerized app installed on OpenRun, a Deployment is created with a Service of type ClusterIP. API calls to the OpenRun API Server are routed to the app specific Service using its cluster IP.

All Kubernetes resources are created lazily, on the first API call to the app. If app is running version 1, and a code/config change is done which updates it to version 2, the deployment update will happen on the next API call to the app. The API call is blocked, the container image is rebuild if required and the deployment is updated using Server Side Apply API calls.

By default, OpenRun will wait to ensure that the app is running pods with only the new version, before processing further API calls. This behavior can be relaxed by setting

```toml {filename="openrun.toml"}
[app_config]
kubernetes.strict_version_check = false
```

in which case OpenRun just makes sure that the new version started successfully,
