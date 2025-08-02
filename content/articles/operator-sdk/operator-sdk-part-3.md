---
title: "Managing Lifecycle and Deployment in Your Operator"
date: 2025-06-09
weight: 3
menu:
  main:
    parent: "articles"
cover:
  image: "/blog/images/operator-sdk/k8s-galleon.jpg"
  relative: true
  hiddenInList: true
draft: false
---

## Managing Lifecycle Events

In this third part of the series, we'll explore how to handle lifecycle events and deploy your Kubernetes operator built with the Operator SDK to a real cluster.

The core of your operator is the **Reconcile loop**. This function is responsible for checking the current state of your custom resource and performing actions to move it toward the desired state.

Open the `internal/controller/magicaloperator_controller.go` and locate the `Reconcile` method. Within this method, you can add logic such as:

- Check if the deployment exists; create it if it doesn't
- Update replicas if the specification has changed
- Log actions for observability and debugging

**For this demonstration**, the <a href="https://github.com/mariobris/operator-sdk-demo/blob/master/internal/controller/magicaloperator_controller.go" target="_blank" rel="noopener noreferrer" style="color:blue;">reconciliation process</a> creates Kubernetes events and logs messages to stdout when a `MagicalOperator` resource is created, updated, or deleted. To create events, don't forget to add a recorder to the <a href="https://github.com/mariobris/operator-sdk-demo/blob/master/cmd/main.go#L150" target="_blank" rel="noopener noreferrer" style="color:blue;">MagicalOperatorReconciler controller</a>.

**Important principle:** Each reconciliation should be **idempotent** — it should safely retry and produce the same result regardless of how many times it's executed.

---

## RBAC Permissions

Ensure you have appropriate RBAC permissions defined in `config/rbac/role.yaml`. This is handled automatically by Kubebuilder, and you can find the relevant `markers` in the controller code. For example:

```go
// +kubebuilder:rbac:groups=wizards.hogwarts.com,resources=magicaloperators,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=wizards.hogwarts.com,resources=magicaloperators/status,verbs=get;update;patch
// +kubebuilder:rbac:groups=wizards.hogwarts.com,resources=magicaloperators/finalizers,verbs=update
// +kubebuilder:rbac:groups="",resources=events,verbs=create;patch
```

Don't forget to run `make manifests` after making changes and check the `rbac` directory for updated files.

📌 **Commit all files at this stage!**

---

## Deploy to Kubernetes

Deployment to Kubernetes uses <a href="https://kustomize.io/" target="_blank" rel="noopener noreferrer" style="color:blue;">Kustomize</a> files in the `config` directory. For example, the target namespace and name prefix can be customized in `config/default/kustomization.yaml`.

Since the image will be built locally, we need to add <a href="https://kubernetes.io/docs/concepts/containers/images/#image-pull-policy" target="_blank" rel="noopener noreferrer" style="color:blue;">imagePullPolicy: IfNotPresent</a> in the `config/manager/manager.yaml` containers specification.

Let's build the container image and deploy it to the Kubernetes cluster:

```
make docker-build
make deploy
```

**Example output:**
```
namespace/demo-operator-system created
customresourcedefinition.apiextensions.k8s.io/magicaloperators.wizards.hogwarts.com created
serviceaccount/demo-operator-controller-manager created
role.rbac.authorization.k8s.io/demo-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/demo-operator-magicaloperator-editor-role created
clusterrole.rbac.authorization.k8s.io/demo-operator-magicaloperator-viewer-role created
clusterrole.rbac.authorization.k8s.io/demo-operator-manager-role created
clusterrole.rbac.authorization.k8s.io/demo-operator-metrics-auth-role created
clusterrole.rbac.authorization.k8s.io/demo-operator-metrics-reader created
rolebinding.rbac.authorization.k8s.io/demo-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/demo-operator-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/demo-operator-metrics-auth-rolebinding created
service/demo-operator-controller-manager-metrics-service created
deployment.apps/demo-operator-controller-manager created
```

Verify that the MagicalOperator is up and running:

```
kubectl get po -n demo-operator-system
NAME                                                READY   STATUS    RESTARTS   AGE
demo-operator-controller-manager-6dd47d7df8-f5jkn   1/1     Running   0          12s
```

### Cleanup (Optional)

When you're done experimenting, you can clean up the resources:

```bash
make undeploy
```

This removes all the resources created by the operator from your cluster.

---

## Conclusion

You've now completed a foundational journey through creating, deploying, and managing a custom Kubernetes operator using the Operator SDK. From understanding the concepts to implementing lifecycle handling and cluster deployment, you're now equipped with the knowledge to build production-ready operators!

### Best Practices for Operator Development

- **Use Fine-Grained RBAC** — Only grant permissions your operator truly needs. Avoid wildcards unless absolutely necessary.
- **Keep CRDs Clean** — Avoid bloating the CRD with unnecessary fields. Document `spec` fields clearly with meaningful descriptions.
- **Handle Errors Gracefully** — Retry on transient errors and update the `status` field to reflect failures appropriately.
- **Implement Proper Observability** — Add structured logging, expose relevant metrics, and track reconciliation errors and retry attempts.
- **Test Locally First** — Always test your operator locally before deploying to a cluster.
- **Version Control** — Commit changes at each step to track your progress and understand what each command does.

### Key Questions to Consider

When developing operators, always ask yourself:

- **Should I use a ConfigMap or a custom resource?** This is well explained in the <a href="https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#should-i-use-a-configmap-or-a-custom-resource" target="_blank" rel="noopener noreferrer" style="color:blue;">official Kubernetes documentation</a>.
- **Which custom resource approach should I use?** Consider <a href="https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#adding-custom-resources" target="_blank" rel="noopener noreferrer" style="color:blue;">CRDs vs API Aggregation</a>. This tutorial focuses on CRDs, which are simpler and can be created without extensive programming.

### Next Steps

Now that you've mastered the basics, consider exploring:
- **Advanced Controller Patterns** — Finalizers, webhooks, and complex reconciliation logic
- **Testing Strategies** — Unit tests, integration tests, and end-to-end testing for operators
- **Metrics and Monitoring** — Implementing custom metrics and integrating with monitoring systems
- **Production Deployment** — Helm charts, OLM (Operator Lifecycle Manager), and CI/CD pipelines

---

Happy automating!
