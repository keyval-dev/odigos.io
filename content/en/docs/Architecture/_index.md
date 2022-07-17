---
title: "Architecture"
linkTitle: "Architecture"
weight: 4
---

{{% alert title="Odigos is still in beta" %}}
Some of the features described in this document may not be available yet. APIs and custom resources may introduce breaking changes.
{{% /alert %}}

## Goals

Odigos acts as a control plane for all the observability needs in a cluster. It is responsible for:

- Automatic instrumentation of applications
- Automatic configuration and deployment of collectors
- Infrastructure observability (Kubernetes nodes observability data)

## High Level Architecture

These tasks are performed by 4 microservices:

- Instrumentor
- Autoscaler
- Scheduler
- UI

The different microservices communicate via Kubernetes API server (see [Custom Resources](/docs/custom-resources) for more details).

The following diagram shows the architecture of the Odigos observability system.

![Odigos Architecture](Odigos_Arch.jpg)

## Instrumentor

The instrumentor microservice is responsible for automatic detection of applications in the cluster and instrumentation of them.
Automatic instrumentation is done according to the applications selected by the user in the UI.
The instrumentor may change the arguments passed to the instrumentation SDK to reflect the following changes:

- A configuration change made by the user (for example changing the sampling rate in the UI)
- Rescheduling done by the scheduler (when the collectors pipeline changes)

### Language Detection

A key part of being able to automatically instrument every new application is to be able to detect the language of the application. After the language is detected Odigos will perform automatic instrumentation according to the language. For runtime languages Odigos uses the appropriate OpenTelemetry instrumentation. For compiled languages Odigos uses eBPF instrumentation. In order to detect the language of the application Odigos deploys a lang detection pod that analyzes one of the target application instances. This pod is deployed on the same node as the target instance and is able to look into the target pod filesystem.

The lang detection pod uses the following heuristics in order to detect the language of the application:

- process name
- environment variables
- dynamically loaded libraries

The lang detection pod reports the detected language by leveraging the `TerminationMessagePath` field of the `Pod` resource.

## Autoscaler

Autoscaler is responsible for deploying and configuring the collectors. Deployment of collectors is done in two scenarios:

- A user action in the UI (for example, adding a new destination)
- A change in observabiltiy traffic (for example, if one of the applications sends most of the data, the autoscaler may decide to deploy a dedicated collector for that application)

## Scheduler

The scheduler service assigns applications discovered by the instrumentor to the collectors pipeline create by the autoscaler.

## UI

Odigos UI is a Next.js application that allows the user to control their observability needs. The UI is not accessible outside of the cluster. In order to access to the UI the user should use port forwarding by executing the following command:

```console
kubectl port-forward svc/odigos-ui 3000:3000 -n odigos-system
```
