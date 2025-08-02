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

- **Compositions** - A template to define how to create resources. "Similar to terraform modules" (quoted).
- **Composite Resource Definition (XRD)** - A custom k8s API specification. "Similar to CRD, e.g. deployment" (quoted).
- **Composite Resource (XR)** - Created by using the custom API defined in a Composite Resource Definition. XRs use the Composition template to create new managed resources.
- **Claims (XRC)** - Like a Composite Resource, but with namespace scoping. Useful for end users, e.g. developers
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

To install the Crossplane function named **function-patch-and-transform**, I'll apply the  <a href="https://github.com/mariobris/crossplane-demo/blob/main/functions.yaml" target="_blank" rel="noopener noreferrer" style="color:blue;">function configuration YAML</a>.

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
- Security Groups
- EC2 Instance

```bash
# VPC and subnets
kubectl apply -f manifests/x/network/composite.yaml
kubectl apply -f manifests/x/network/composition.yaml
# Compute and SGs
kubectl apply -f manifests/x/compute/composite.yaml
kubectl apply -f manifests/x/compute/composition.yaml
```

## Claims and Namespace-Scoped Resources

Claims provide namespace-scoped access to composite resources, making them ideal for end users like developers.

### Claim Benefits

- **Namespace Isolation**: Resources are scoped to specific namespaces
- **User-Friendly**: Simpler API for end users
- **RBAC Integration**: Leverage Kubernetes RBAC for access control
- **Self-Service**: Enable developers to provision infrastructure safely

I'll apply the composition claim
```bash
kubectl apply -f manifests/x/network/claim.yaml
kubectl apply -f manifests/x/compute/claim.yaml
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

## Best Practices for Production

1. **RBAC Configuration**: Implement proper role-based access control
2. **Secret Management**: Secure sensitive configuration
3. **Network Policies**: Restrict network access to Crossplane components
4. **Audit Logging**: Enable audit logs for compliance

## What's Next?

This concludes our Crossplane tutorial series! You now have a solid foundation in:

- Understanding Crossplane concepts and architecture
- Working with providers and managed resources
- Building compositions and composite resources
- Troubleshooting common issues

To continue your Crossplane journey, explore the additional resources below and consider diving into advanced topics like:
- Production deployment strategies
- Multi-cloud infrastructure management
- Advanced composition patterns
- Crossplane community resources

## Additional Resources

- <a href="https://docs.crossplane.io/latest/concepts/compositions/" target="_blank" rel="noopener noreferrer" style="color:blue;">Compositions Documentation</a>
- <a href="https://docs.crossplane.io/latest/concepts/composite-resources/" target="_blank" rel="noopener noreferrer" style="color:blue;">Composite Resources Guide</a>
- <a href="https://docs.crossplane.io/latest/concepts/claims/" target="_blank" rel="noopener noreferrer" style="color:blue;">Claims Documentation</a>
- <a href="https://docs.crossplane.io/latest/concepts/functions/" target="_blank" rel="noopener noreferrer" style="color:blue;">Functions Guide</a>
- <a href="https://docs.crossplane.io/latest/concepts/functions/" target="_blank" rel="noopener noreferrer" style="color:blue;">Functions Documentation</a>
- <a href="https://docs.upbound.io/operate/simulations/" target="_blank" rel="noopener noreferrer" style="color:blue;">Simulation Guide</a>
