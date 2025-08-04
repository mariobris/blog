---
title: "Working with Providers and Managed Resources"
date: 2025-08-02
weight: 12
menu:
  main:
    parent: "articles"
cover:
  relative: true
  hiddenInList: true
draft: false
ShowPostNavLinks: true
---

## Installation

I will use k3s for this demo. Follow the <a href="https://docs.k3s.io/quick-start#install-script" target="_blank" rel="noopener noreferrer" style="color:blue;">official installation guide</a> to install k3s.

I will install Crossplane in my Kubernetes cluster (k3s):

```bash
# Add the Crossplane Helm repository
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

# Install Crossplane
helm upgrade -i crossplane \
  --namespace crossplane-system \
  --create-namespace \
  crossplane-stable/crossplane

# Set the context to crossplane-system namespace
kubectl config set-context --namespace default --current
```

Verify the installation:

```bash
kubectl get pods -n crossplane-system
```

I should see pods like `crossplane-*` and `crossplane-rbac-manager-*` running.

## Installing AWS Family Providers

AWS family providers are specialized providers for different AWS services, providing better resource coverage and more focused functionality. I will install them:

First, I'll clone the demo repository to get the required YAML files:

```bash
git clone https://github.com/mariobris/crossplane-demo
cd crossplane-demo
```

Then apply the AWS family providers:

```bash
kubectl apply -f provider-aws.yaml
```

The <a href="https://github.com/mariobris/crossplane-demo/blob/main/provider-aws.yaml" target="_blank" rel="noopener noreferrer" style="color:blue;">provider-aws.yaml</a> file defines the AWS family providers needed for this demo. It includes separate providers for S3, EC2, and IAM. Each provider manages only its specific AWS service, which helps keep things simple and focused.

Check installed API resources:

```bash
kubectl api-resources --categories=crossplane
```

## Provider Configuration

To allow the AWS family providers to communicate with AWS services, I must configure authentication. This is done using a ProviderConfig resource, which tells the providers how to authenticate with AWS.

### AWS Authentication Setup

**Note:** I need valid AWS credentials with sufficient permissions to work with AWS resources. For this demo, I'm using an admin role with full access to AWS services.

1. Create AWS credentials file:

```bash
cat > aws-credentials.txt << EOF
[default]
aws_access_key_id = <your-access-key>
aws_secret_access_key = <your-secret-key>
EOF
```

2. Create a Kubernetes secret:

```bash
kubectl create secret generic aws-creds \
  --namespace crossplane-system \
  --from-file=secret-key=./aws-credentials.txt
```

3. Apply the provider configuration:

```bash
kubectl apply -f providerConfig.yaml
```

**Note:** This provider configuration is shared across all AWS family providers (S3, EC2, IAM, etc.).

## Working with Managed Resources

Managed resources represent external cloud services as Kubernetes objects. Let's explore how to work with different AWS resources using Crossplane.

### Your First Managed Resource

I will create a simple S3 bucket to test the setup using the AWS S3 family provider. I can find the example manifest <a href="https://github.com/mariobris/crossplane-demo/blob/main/manifests/simple/s3.yaml" target="_blank" rel="noopener noreferrer" style="color:blue;">here</a>.

**Important:** S3 bucket names must be globally unique. Before applying the manifest, I'll open <a href="https://github.com/mariobris/crossplane-demo/blob/main/manifests/simple/s3.yaml" target="_blank" rel="noopener noreferrer" style="color:blue;">manifests/simple/s3.yaml</a> and change the `Name` field of the resource to a unique value that is not already used by anyone else.

Apply the manifest:
```bash
kubectl apply -f manifests/simple/s3.yaml
```

Check the status:

```bash
kubectl get bucket
```

I can see that the bucket is `SYNCED` and `READY`. If it's not, then follow the [troubleshooting section](#troubleshooting).

**Note:** This example uses the `provider-aws-s3` family provider, which provides specialized S3 functionality.

### Creating Multiple Resources

I will create common AWS resources and observe their state:

```bash
kubectl apply -f manifests/simple/
```

Observe resources:

```bash
kubectl get managed
```

I will list the secrets containing connection details for the S3 bucket:

```bash
kubectl get bucket
kubectl get secrets -n crossplane-system --field-selector type=connection.crossplane.io/v1alpha1
```

Delete resources:

```bash
kubectl delete -f manifests/simple/
```

## Helm Templating

Simple manifests can be deployed with Kustomize or Helm. I will deploy the same resources with Helm now:

```bash
helm upgrade -i -n default simple manifests/helm/
```

Observe the created resources and review the <a href="https://github.com/mariobris/crossplane-demo/blob/main/manifests/helm/values.yaml" target="_blank" rel="noopener noreferrer" style="color:blue;">values.yaml</a> file:

```bash
kubectl get managed
```

Delete resources:

```bash
helm uninstall -n default simple
```

## Resource Dependencies

Crossplane handles resource dependencies through **selectors**. For example, when creating an EC2 instance, you might need:

1. VPC
2. Subnet (with VPC selector)
3. Instance (with subnet selectors)
4. InstanceState (with instance selectors)

Crossplane uses selectors like `vpcIdSelector` and `subnetIdSelector` to reference other resources to pass values between resources. The creation order is determined by these explicit references, not automatic dependency detection.

### Example: Using Selectors

First, create a VPC with the required label:

```yaml
apiVersion: ec2.aws.upbound.io/v1beta1
kind: VPC
metadata:
  name: demo-vpc
  labels:
    app: demo-vpc
spec:
  forProvider:
    region: us-east-1
    cidrBlock: 192.168.0.0/16
  providerConfigRef:
    name: user-keys
```

Then create a Subnet that references the VPC using a selector:

```yaml
apiVersion: ec2.aws.upbound.io/v1beta1
kind: Subnet
metadata:
  name: demo-subnet
spec:
  forProvider:
    vpcIdSelector:
      matchLabels:
        app: demo-vpc
    cidrBlock: 192.168.0.0/24
    region: us-east-1
  providerConfigRef:
    name: user-keys
```

**Note:** The `vpcIdSelector` tells Crossplane to find the VPC resource with the label `app: demo-vpc` and use its ID for the subnet's VPC reference.

## Best Practices

### Resource Naming

Use consistent Kubernetes naming conventions:

```yaml
metadata:
  name: ${environment}-${service}-${resource-type}-${identifier}
```

### Tagging Strategy

Implement consistent AWS tagging:

```yaml
tags:
  Name: ${resource-name}
  Environment: ${environment}
  Project: ${project}
  Owner: ${team}
```

### Resource Organization

Organize resources by environment or project using namespaces or labels.

## Troubleshooting

When working with Crossplane, you may encounter various issues. Here's a systematic approach to diagnose and resolve common problems:

### 1. Komoplane

See the [Komoplane section](./crossplane-part-1.md#komoplane)

### 2. Check Provider Status

First, verify that your providers are properly installed and healthy:

```bash
kubectl get -n crossplane-system provider
kubectl describe -n crossplane-system provider <provider-name>
```

**Why:** Providers must be healthy for Crossplane to manage external resources. If a provider is not ready, resource creation will fail.

### 3. View Provider Logs

Check provider logs for detailed error messages:

```bash
kubectl get -n crossplane-system pods
kubectl logs -n crossplane-system <pod-name>
```

**Why:** Provider logs contain detailed information about authentication failures, API errors, and resource creation issues.

### 4. Monitor Resource Status

Check the status of your managed resources:

```bash
# Check resource status
kubectl get managed

# Check specific resource status
kubectl describe <managed-resource-name>

# Check resource events
kubectl get events --sort-by='.lastTimestamp' -A
```

**Why:** Check resource status, get detailed error messages, and monitor events for troubleshooting.

## Additional Resources

- <a href="https://github.com/mariobris/crossplane-demo" target="_blank" rel="noopener noreferrer" style="color:blue;">Crossplane Demo Repository</a> - Contains all the YAML files used in this tutorial
