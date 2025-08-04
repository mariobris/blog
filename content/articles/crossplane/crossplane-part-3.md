---
title: "Compositions and Composite Resources"
date: 2025-08-02
weight: 13
menu:
  main:
    parent: "articles"
cover:
  relative: true
  hiddenInList: true
draft: false
ShowPostNavLinks: true
---


## Core Concepts

- **Compositions** - A template to define how to create resources (similar to Terraform modules)
- **Composite Resource Definition (XRD)** - A custom Kubernetes API specification (similar to CRD, e.g., Deployment)
- **Composite Resource (XR)** - Created by using the custom API defined in a Composite Resource Definition. XRs use the Composition template to create new managed resources
- **Claims (XRC)** - Like a Composite Resource, but with namespace scoping. Useful for end users, e.g., developers
- **Functions** - Crossplane extensions that template resources and allow custom logic during composition

### Composition
Compositions are templates for creating multiple managed resources as a single object. A Composition composes individual managed resources together into a larger, reusable solution.

**Think of it as:** Similar to Terraform modules - a template that defines how to create resources.

Compositions provide several benefits:
- **Reusability**: Define infrastructure patterns once, use them everywhere
- **Consistency**: Ensure all environments follow the same infrastructure patterns
- **Abstraction**: Hide complexity behind simple, declarative APIs
- **Governance**: Enforce policies and standards across all deployments

### Composite Resource Definition (XRD)
A custom Kubernetes API specification, similar to a CRD (like a Deployment), but with additional options related to Crossplane, such as Claims and connection secrets.

### Composite Resource (XR)
A composite resource represents a set of managed resources as a single Kubernetes object. Crossplane creates composite resources when users access a custom API, defined in the CompositeResourceDefinition.

### Claim (XRC)
Claims are like composite resources, but with namespace scoping. Useful for end users, such as developers.

**Key Difference:** Crossplane can create Claims in a namespace, while composite resources are cluster-scoped.

## Functions

Composition functions (or just functions, for short) are Crossplane extensions that template Crossplane resources. You can browse available functions in the <a href="https://marketplace.upbound.io/functions" target="_blank" rel="noopener noreferrer" style="color:blue;">Upbound Functions Marketplace</a>. Crossplane calls these composition functions to determine what resources it should create when you create a composite resource (XR).

### Using Functions in Compositions

Functions can be used to:
- Transform data between resources
- Validate inputs
- Generate dynamic values
- Implement complex business logic

To install the Crossplane function named **function-patch-and-transform**, I'll apply the <a href="https://github.com/mariobris/crossplane-demo/blob/main/functions.yaml" target="_blank" rel="noopener noreferrer" style="color:blue;">functions.yaml</a>:

```bash
kubectl apply -f functions.yaml
```

#### Field Patching

Patch values from the composite resource to managed resources:

```yaml
patches:
  - fromFieldPath: spec.region
    toFieldPath: spec.forProvider.region
```

#### Transforms

Transform values during patching:

```yaml
patches:
  - fromFieldPath: metadata.name
    toFieldPath: spec.forProvider.tags.Name
    transforms:
      - type: string
        string:
          fmt: "%s-vpc"
```

## Simulation

Crossplane supports simulation mode to preview changes before applying them—similar to Terraform's `plan` command. You can learn more and see examples in the <a href="https://docs.upbound.io/operate/simulations/" target="_blank" rel="noopener noreferrer" style="color:blue;">official simulation guide</a>. This allows you to see what changes would be made without actually applying them.

## Building Your First Composition

I will create a simple web application infrastructure composition that includes:
- VPC
- Subnet
- EC2 Instance
- S3 bucket

```bash
# VPC and subnets
kubectl apply -f manifests/x/network/xrd.yaml
kubectl apply -f manifests/x/network/composition.yaml
# Compute and S3
kubectl apply -f manifests/x/compute/xrd.yaml
kubectl apply -f manifests/x/compute/composition.yaml
```

Then I will check the resources:

```bash
# XRDs
kubectl get xrd
# Compositions
kubectl get compositions
```

## Claims

Claims provide namespace-scoped access to composite resources, making them ideal for end users like developers.

### Claim Benefits

- **Namespace Isolation**: Resources are scoped to specific namespaces
- **User-Friendly**: Simpler API for end users
- **RBAC Integration**: Leverage Kubernetes RBAC for access control
- **Self-Service**: Enable developers to provision infrastructure safely

I'll first apply the composition claim for network:

```bash
kubectl apply -f manifests/x/network/claim.yaml
```

And validate the state:

```bash
# Composite
kubectl get xnetwork
# Claim
kubectl get network
```

Once the network is `SYNCED` and `READY`, then I will apply claim for compute:

```bash
kubectl apply -f manifests/x/compute/claim-x.yaml
```

And validate the state:

```bash
# Composite
kubectl get xcompute
# Claim
kubectl get compute
```

## Troubleshooting Compositions

When working with compositions, composite resources, and claims, you may encounter specific issues. Here's how to diagnose and resolve them:

### 1. Check XRD Status

First, verify that your Composite Resource Definition is properly installed:

```bash
kubectl get xrd
kubectl describe xrd <xrd-name>
```

**Why:** XRDs define the API for composite resources. If XRD is not ready, you cannot create composite resources or claims.

### 2. Verify Composition Status

Check that your compositions are properly configured:

```bash
kubectl get composition
kubectl describe composition <composition-name>
```

**Why:** Compositions define how resources are created. If composition is not ready, composite resources cannot be created.

### 3. Monitor Composite Resources

Check the status of your composite resources:

```bash
kubectl get composite
kubectl describe composite <composite-name>
```

**Why:** Composite resources show the status of the composed managed resources. This helps identify which resources failed to create.

### 4. Check Claims Status

If using claims, verify their status:

```bash
kubectl get claim
kubectl describe claim <claim-name>
```

**Why:** Claims are namespace-scoped composite resources. Their status indicates whether the underlying composite resource was created successfully.

### 5. Verify Function Status

If using composition functions, check their status:

```bash
kubectl get function
kubectl describe function <function-name>
```

**Why:** Functions process composition logic. If a function is not ready, compositions may fail to create resources.

## Development Tools

For a better development experience when working with compositions and XRDs, I recommend installing the <a href="https://github.com/upbound/vscode-up" target="_blank" rel="noopener noreferrer" style="color:blue;">VS Code Extension for Crossplane</a>. This extension provides diagnostics as you work through its integration with xpls:

- Crossplane.yaml dependency version validation
- Crossplane.yaml dependency type validation
- Crossplane.yaml dependency missing validation
- XRC schema validation
- Composed resource schema validation
- Composed resource schema validation with patched details
- XRD openAPIv3Schema validation


## Additional Resources

- <a href="https://docs.crossplane.io/latest/concepts/compositions/" target="_blank" rel="noopener noreferrer" style="color:blue;">Compositions Documentation</a>
- <a href="https://docs.crossplane.io/latest/concepts/composite-resources/" target="_blank" rel="noopener noreferrer" style="color:blue;">Composite Resources Guide</a>
- <a href="https://docs.crossplane.io/latest/concepts/claims/" target="_blank" rel="noopener noreferrer" style="color:blue;">Claims Documentation</a>
- <a href="https://docs.crossplane.io/latest/concepts/functions/" target="_blank" rel="noopener noreferrer" style="color:blue;">Functions Guide</a>
- <a href="https://docs.crossplane.io/latest/concepts/functions/" target="_blank" rel="noopener noreferrer" style="color:blue;">Functions Documentation</a>
- <a href="https://docs.upbound.io/operate/simulations/" target="_blank" rel="noopener noreferrer" style="color:blue;">Simulation Guide</a>
- <a href="https://github.com/upbound/vscode-up" target="_blank" rel="noopener noreferrer" style="color:blue;">VS Code Extension</a>


## Conclusion

Throughout this tutorial series, I've demonstrated how Crossplane enables you to manage cloud infrastructure using Kubernetes-native APIs. You've learned to install providers, create managed resources, build compositions, and implement claims for self-service infrastructure.

Crossplane provides several key security advantages:

**Kubernetes RBAC Integration**: Leverage existing Kubernetes role-based access control to manage who can create, modify, or delete infrastructure resources. This eliminates the need for separate IAM policies and provides fine-grained access control.

**Secret Management**: Crossplane automatically stores sensitive connection details in Kubernetes secrets, ensuring credentials are encrypted at rest and follow Kubernetes security best practices. This prevents credential sprawl and centralizes secret management.

**Audit Trail**: All infrastructure changes are logged through Kubernetes audit logs, providing comprehensive visibility into who made changes, when, and what resources were affected. This enables compliance and security monitoring.

Crossplane excels in environments where you're already heavily invested in Kubernetes and want to extend that ecosystem to infrastructure management.
