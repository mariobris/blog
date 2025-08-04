---
title: "Getting Started with Crossplane"
date: 2025-08-02
weight: 11
menu:
  main:
    parent: "articles"
cover:
  relative: true
  hiddenInList: true
draft: false
ShowPostNavLinks: true
---

## What Is Crossplane?

<a href="https://www.crossplane.io/" target="_blank" rel="noopener noreferrer" style="color:blue;">Crossplane</a> is a cloud-native control plane that enables you to manage infrastructure across multiple cloud providers using Kubernetes-native APIs. Think of it as a universal interface for cloud resources that extends Kubernetes' declarative model to external infrastructure.

Crossplane allows you to:
- Manage cloud resources using Kubernetes manifests
- Create reusable infrastructure templates
- Implement GitOps workflows for infrastructure
- Maintain consistent APIs across different cloud providers
- Automate infrastructure provisioning and management

Crossplane is particularly valuable for organizations that need to manage infrastructure across multiple cloud providers or want to provide self-service infrastructure to development teams through familiar Kubernetes APIs.

## Core Concepts

### Managed Resource (MR)
A managed resource represents an external service in a Provider. Crossplane calls the object inside Kubernetes a managed resource and the external object inside the Provider an external resource.

**Examples:**
- Amazon AWS EC2 Instance
- Google Cloud GKE Cluster
- Microsoft Azure PostgreSQL Database

**Key Point:** Ingress object in Kubernetes is a Custom Resource, while an EC2 instance in Kubernetes is a Managed Resource.

### AWS Providers

Providers in Crossplane are like installing a Kubernetes operator for each cloud resource type. They define the Custom Resource Definitions (CRDs) that let you manage external resources as Kubernetes objects.

For AWS, there are several provider options:

1. **AWS Family Providers**: Specialized providers for specific AWS services (recommended). See the <a href="https://marketplace.upbound.io/providers?query=aws" target="_blank" rel="noopener noreferrer" style="color:blue;">AWS Provider Marketplace</a> for available options.
2. **Provider AWS (Upbound)**: <a href="https://marketplace.upbound.io/providers?query=aws" target="_blank" rel="noopener noreferrer" style="color:blue;">Official Upbound AWS Provider</a>
3. **Provider Upjet AWS**: <a href="https://github.com/crossplane-contrib/provider-upjet-aws" target="_blank" rel="noopener noreferrer" style="color:blue;">Community Upjet AWS Provider</a>
4. **Provider AWS (Crossplane Contrib)**: <a href="https://github.com/crossplane-contrib/provider-aws" target="_blank" rel="noopener noreferrer" style="color:blue;">Community AWS Provider</a>

#### AWS Authentication Methods

AWS provider supports several authentication methods for Crossplane providers:

1. **Static Credentials**: Use AWS access keys (recommended for local development or demos).
2. **IAM Roles**: Use IAM roles attached to EKS or EC2 instances.
3. **Web Identity**: Use OIDC-based authentication.
4. **Service Account (IRSA)**: Use IAM Roles for Service Accounts, typically in EKS.

For detailed instructions on each method, refer to the <a href="https://docs.upbound.io/providers/provider-aws/authentication" target="_blank" rel="noopener noreferrer" style="color:blue;">AWS Provider Authentication Guide</a>.

**Note:** In this demo, since I'm running on k3s, I will use static credentials.

## Connection Details

Connection details automatically store sensitive information from managed resources into Kubernetes secrets. This allows applications to access credentials, endpoints, and other connection data securely.

Common connection details include:
- Database connection strings
- API endpoints and credentials
- Access keys and secret keys
- Resource ARNs and IDs
- Network endpoints and ports

Example: S3 bucket connection details at the bottom of <a href="https://github.com/mariobris/crossplane-demo/blob/main/manifests/simple/s3.yaml" target="_blank" rel="noopener noreferrer" style="color:blue;">s3.yaml</a> file.

### How Connection Details Work

When you create a managed resource with connection details configured, Crossplane automatically:
1. Extracts specified fields from the resource's status
2. Creates a Kubernetes secret with the extracted values
3. Makes the secret available for other resources or applications to consume

### Benefits of Connection Details

- **Security**: Sensitive data is stored in Kubernetes secrets
- **Automation**: No manual credential management
- **Integration**: Easy to connect resources and applications
- **Audit Trail**: All access is logged through Kubernetes
- **RBAC**: Leverage Kubernetes RBAC for access control

## Komoplane

Once you have Kubernetes running and Crossplane installed, you can use a web UI called <a href="https://github.com/komodorio/komoplane" target="_blank" rel="noopener noreferrer" style="color:blue;">Komoplane</a> to visualize your Crossplane resources.

Komoplane helps you see all Crossplane objects in your cluster in a simple web interface. It is useful for understanding what resources exist and how they are connected.

To use Komoplane from your local machine (Linux):

1. Download the latest Komoplane binary for your operating system from the <a href="https://github.com/komodorio/komoplane/releases" target="_blank" rel="noopener noreferrer" style="color:blue;">releases page</a>.

2. Extract the downloaded file (if compressed) and make the binary executable:
   ```bash
   chmod +x komoplane
   ```

3. Set your Kubernetes context to the desired cluster:
   ```bash
   kubectl config use-context your-cluster
   ```

4. Run Komoplane:
   ```bash
   ./komoplane
   ```

Komoplane will start a local web server for you to access the UI in your browser.

5. Access the UI: Open `http://localhost:8090` in your browser


## Additional Resources

- <a href="https://docs.crossplane.io/latest/concepts/" target="_blank" rel="noopener noreferrer" style="color:blue;">Crossplane Concepts Documentation</a>
- <a href="https://docs.crossplane.io/latest/getting-started/install-configure/" target="_blank" rel="noopener noreferrer" style="color:blue;">Installation Guide</a>
- <a href="https://docs.crossplane.io/latest/concepts/providers/" target="_blank" rel="noopener noreferrer" style="color:blue;">Crossplane Providers Concepts</a>
- <a href="https://marketplace.upbound.io/providers" target="_blank" rel="noopener noreferrer" style="color:blue;">Provider Marketplace</a>
- <a href="https://docs.upbound.io/providers" target="_blank" rel="noopener noreferrer" style="color:blue;">Providers Documentation</a>
- <a href="https://docs.upbound.io/providers/provider-aws/" target="_blank" rel="noopener noreferrer" style="color:blue;">AWS Provider Documentation</a>
- <a href="https://docs.upbound.io/providers/provider-aws/authentication/" target="_blank" rel="noopener noreferrer" style="color:blue;">AWS Authentication Guide</a>
- <a href="https://docs.crossplane.io/latest/concepts/managed-resources/" target="_blank" rel="noopener noreferrer" style="color:blue;">Managed Resources Documentation</a>
- <a href="https://github.com/crossplane/crossplane" target="_blank" rel="noopener noreferrer" style="color:blue;">Crossplane GitHub Repository</a>
