# Extension Proposal: Automated GitOps Deployment with Flux

## Identified Shortcoming

A critical release-engineering shortcoming of the project was the **manual and imperative deployment process** for Kubernetes resources. Application deployments, infrastructure components (Istio, monitoring), and image updates were applied manually using Helm and `kubectl`, with no enforced order or automation.

This made deployments **error-prone and difficult to reproduce**. Common issues included missing CRDs, incorrect deployment order, and outdated image references. Recreating a working cluster required manual intervention and detailed knowledge of deployment steps, which is undesirable for a release pipeline.

This shortcoming is directly related to the *deployment and release engineering* aspects of the assignment.

---

## Effect on the Project

The manual deployment approach led to:
- fragile releases that could fail due to missing prerequisites,
- inconsistent environments across machines,
- high cognitive load during deployment,
- slow iteration when testing new image versions.

As the project grew in complexity, these issues became increasingly disruptive.

---

## Proposed Extension

To address this, we propose and implement a **GitOps-based deployment workflow using Flux**.

Flux continuously reconciles the desired cluster state from Git, making Git the single source of truth for both infrastructure and application deployments. This removes the need for manual deployment steps and enforces correct ordering and consistency.

---

## Implemented Solution

The extension is not only conceptual but **fully implemented**:

- A [**Flux repository**](https://github.com/doda2025-team20/team20-flux) defines:
    - infrastructure components (Istio, monitoring),
    - application HelmReleases,
    - image automation resources.
- Applications are deployed using **Flux HelmRelease CRDs** instead of manual Helm commands.
- **Flux image automation** updates Helm values automatically when new container images are published.
- A [**Vagrant + Ansible playbook**](https://github.com/doda2025-team20/operation/blob/main/infra/playbooks/flux.yaml) installs Flux on the controller node and bootstraps the Git repository, enabling one-command cluster setup.

This implementation is realistic and achievable within a few days of effort.

An important property of this setup is that Flux continuously mirrors the desired state defined in Git into the Kubernetes cluster. Any change committed to the GitOps repository—such as updating a HelmRelease, changing configuration values, or modifying image references—is automatically detected and reconciled by Flux within a short interval. This ensures that the cluster state always converges toward the version-controlled configuration, and that changes can be reviewed, audited, and reverted using standard Git workflows.

---

## Expected Outcome

With this extension:
- deployments become reproducible and declarative,
- manual image updates are eliminated,
- deployment order is enforced automatically,
- rebuilding a cluster from scratch becomes straightforward.

Overall, the release process becomes more reliable, scalable, and maintainable.

---

## Evaluation

The effectiveness of the extension can be evaluated by:
- comparing the number of manual deployment steps before and after,
- measuring deployment time for new image versions,
- rebuilding the cluster on a fresh setup and verifying correctness.

A reduction in errors and manual steps demonstrates the success of the change.

---

## Assumptions and Downsides

The approach assumes familiarity with Git and Kubernetes declarative workflows. GitOps introduces additional tooling and a learning curve, but these costs are outweighed by the improvements in reliability and automation.

---

## References

- Flux Documentation: https://fluxcd.io/docs/
- GitOps Principles: https://about.gitlab.com/topics/gitops/
