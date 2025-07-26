---
title: "Getting Started with Operator SDK"
date: 2025-06-07
weight: 2
menu:
  main:
    parent: "articles"
cover:
  image: "/blog/images/operator-sdk/k8s-galleon.jpg"
  relative: true
  hiddenInList: true
draft: false
# description: "Desc Text."
# hideSummary: false
# searchHidden: false
# ShowWordCount: true
# UseHugoToc: true
ShowPostNavLinks: true
---

## What Is a Kubernetes Operator?

Kubernetes Operators are a powerful way to automate the ongoing operational tasks involved in running complex applications on Kubernetes.

Rather than relying solely on static configuration files (such as Pods and ConfigMaps), Operators introduce intelligent controllers that understand how your applications should behave. Think of an Operator as having a built-in Site Reliability Engineer (SRE) within your cluster — continuously monitoring and responding to changes.

Operators enable you to automate critical tasks such as:
- Application provisioning and configuration
- Health monitoring and automated remediation
- Handling upgrades and rollbacks
- Managing backups and data restoration
- Intelligent scaling based on application-specific requirements

Operators are particularly valuable for managing **stateful, distributed applications** (such as databases, caches, and message queues), where orchestration requires more sophisticated logic than simple YAML deployment. They enable teams to codify deep operational knowledge into software that runs within the cluster and responds to real-world conditions — just as a human operator would.

## Getting Started with Operator SDK

In this guide, you'll learn how to build a functional Kubernetes operator from scratch using the <a href="https://sdk.operatorframework.io/" target="_blank" rel="noopener noreferrer" style="color:blue;">Operator SDK</a>. This tutorial focuses on helping you understand the essential building blocks and minimal coding required to get a basic operator up and running. While other tools like Kubebuilder exist, we use Operator SDK for its widespread adoption and ease of use.

### Why Choose Operator SDK?

The Operator SDK is a framework that is part of the <a href="https://landscape.cncf.io/" target="_blank" rel="noopener noreferrer" style="color:blue;">CNCF Landscape</a>, which showcases a wide range of cloud-native projects. It leverages the controller-runtime library to streamline operator development by providing:

- High-level APIs and abstractions for common patterns
- Built-in support for CRDs, RBAC, and webhooks
- Tools to scaffold, build, test, and deploy operators
- Extensions to cover common operator use cases

With Operator SDK, you can create robust, production-grade operators more efficiently with reduced manual configuration.

---

## Essential Concepts to Review

Before diving into development, it's recommended to review the following Kubernetes concepts, as this guide will not cover low-level Kubernetes details:

- <a href="https://kubernetes.io/docs/concepts/overview/kubernetes-api/" target="_blank" rel="noopener noreferrer" style="color:blue;">Kubernetes API Concepts</a>
- <a href="https://kubernetes.io/docs/concepts/overview/components/" target="_blank" rel="noopener noreferrer" style="color:blue;">Kubernetes Components</a>
- <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/" target="_blank" rel="noopener noreferrer" style="color:blue;">Kubernetes Objects</a>
- <a href="https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/" target="_blank" rel="noopener noreferrer" style="color:blue;">Custom Resource Definitions (CRDs)</a>
- <a href="https://kubernetes.io/docs/reference/access-authn-authz/rbac/" target="_blank" rel="noopener noreferrer" style="color:blue;">Role-Based Access Control (RBAC)</a>

**Tools Used by the Operator SDK Framework:**
- <a href="https://go.dev/" target="_blank" rel="noopener noreferrer" style="color:blue;">Go Programming Language</a>
- <a href="https://book.kubebuilder.io/" target="_blank" rel="noopener noreferrer" style="color:blue;">Kubebuilder</a>
- <a href="https://book.kubebuilder.io/reference/markers.html" target="_blank" rel="noopener noreferrer" style="color:blue;">Kubebuilder Markers</a>
- <a href="https://kustomize.io/" target="_blank" rel="noopener noreferrer" style="color:blue;">Kustomize</a>
- <a href="https://prometheus.io/docs/instrumenting/exporters/" target="_blank" rel="noopener noreferrer" style="color:blue;">Prometheus Exporters</a>
- <a href="https://sdk.operatorframework.io/docs/overview/cheat-sheet/" target="_blank" rel="noopener noreferrer" style="color:blue;">Operator Framework Cheat Sheet</a>

**Real-World Examples:**
- <a href="https://github.com/prometheus-operator/kube-prometheus" target="_blank" rel="noopener noreferrer" style="color:blue;">kube-prometheus-stack</a>
- <a href="https://github.com/minio/operator" target="_blank" rel="noopener noreferrer" style="color:blue;">MinIO Operator</a>

---

## Understanding CRDs and API Resources

**Custom Resource Definition (CRD):** A mechanism for extending Kubernetes by defining new resource types. It's essentially a way to tell Kubernetes about a new kind of object it should understand — for example, while Kubernetes natively knows about Pods and Services, a CRD allows you to add your own resource types, such as "MyDatabase".

**API Resource:** Any object in Kubernetes that you can create, read, update, or delete — including built-in resources like Pods and Services, as well as custom resources defined through CRDs.

Let's explore practical examples of CRDs and API aggregation:

```bash
kubectl get apiservices
```

Example output:
```bash
NAME                                   SERVICE                      AVAILABLE   AGE
v1alpha1.traefik.io                    Local                        True        117d
v1beta1.metrics.k8s.io                 kube-system/metrics-server   True        117d
```

In this output, you can observe services like `Local` and `kube-system/metrics-server`. This indicates that Traefik uses Kubernetes API extension, while the Metrics Server employs API aggregation through its dedicated `metrics-server` API service running in the `kube-system` namespace.

Kubernetes custom resources can be either **namespaced** (existing within a specific namespace) or **cluster-scoped** (available across the entire cluster). You can inspect them using:

```bash
kubectl api-resources
```

Example output:
```bash
NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND
networkpolicies                     netpol       networking.k8s.io/v1              true         NetworkPolicy
runtimeclasses                                   node.k8s.io/v1                    false        RuntimeClass
```

**Field Descriptions:**

`NAME`
: The resource name

`SHORTNAMES`
: Short aliases for the resource

`APIVERSION`
: The API group and version

`NAMESPACED`
: `true` if the resource is namespaced, `false` if cluster-scoped

`KIND`
: The resource kind

<p></p> <!-- end of definition list -->

Kubernetes serves both OpenAPI v2.0 and OpenAPI v3.0 specifications. [OpenAPI v3](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#openapi-v3) is the preferred and recommended method.

Let's examine how a CRD (OpenAPIV3Schema) is structured:

```bash
kubectl get crds -o yaml traefikservices.traefik.io | head -n 50
```

Example output:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.14.0
    meta.helm.sh/release-name: traefik-crd
    meta.helm.sh/release-namespace: kube-system
  creationTimestamp: "2025-01-29T14:42:00Z"
  generation: 1
  labels:
    app.kubernetes.io/managed-by: Helm
  name: traefikservices.traefik.io
  resourceVersion: "505"
  uid: 04d19218-188f-43da-8dd0-4aa598d684ec
spec:
  conversion:
    strategy: None
  group: traefik.io
  names:
    kind: TraefikService
    listKind: TraefikServiceList
    plural: traefikservices
    singular: traefikservice
  scope: Namespaced
  versions:
  - name: v1alpha1
    schema:
      openAPIV3Schema:
        description: |-
          TraefikService is the CRD implementation of a Traefik Service.
          TraefikService object allows to:
          - Apply weight to Services on load-balancing
          - Mirror traffic on services
          More info: https://doc.traefik.io/traefik/v2.11/routing/providers/kubernetes-crd/#kind-traefikservice
        properties:
          apiVersion:
            description: |-
              APIVersion defines the versioned schema of this representation of an object.
              Servers should convert recognized schemas to the latest internal value, and
              may reject unrecognized values.
              More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
            type: string
          kind:
            description: |-
              Kind is a string value representing the REST resource this object represents.
              Servers may infer this from the endpoint the client submits requests to.
              Cannot be updated.
              In CamelCase.
              More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
            type: string
```

While this output is quite verbose, you can use `kubectl explain` for a more readable format:

```bash
kubectl explain traefikservices.traefik.io | head -n 50
```

Example output:
```bash
GROUP:      traefik.io
KIND:       TraefikService
VERSION:    v1alpha1

DESCRIPTION:
    TraefikService is the CRD implementation of a Traefik Service.
    TraefikService object allows to:
    - Apply weight to Services on load-balancing
    - Mirror traffic on services
    More info:
    https://doc.traefik.io/traefik/v2.11/routing/providers/kubernetes-crd/#kind-traefikservice

FIELDS:
  apiVersion	<string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind	<string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata	<ObjectMeta> -required-
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec	<Object> -required-
    TraefikServiceSpec defines the desired state of a TraefikService.
```

This complexity might lead you to ask: **"How can I create CRDs correctly without making mistakes?"**

**Excellent news:** You don't need to handle this manually — the Operator SDK framework handles the heavy lifting for you!

---

## What's Next

Now that you have a solid understanding of the Operator SDK and relevant Kubernetes concepts, you're ready to proceed to the practical implementation in the next section!
