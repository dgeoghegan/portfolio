# Portfolio Overview — Denis Geoghegan

This page is a guide to two constrained infrastructure demos, each designed so its core claims can be verified through read-only inspection in a few minutes.

If you only have **five minutes**, start with the **GitOps release management demo** below.

These demos are reproducible from scratch and were last verified working in **February 2026**. They are not permanently running services.

---

## GitOps Release Management  
*(three repositories: application, release controller, infrastructure)*

This demo implements a minimal GitOps-based release system for a versioned application running on Kubernetes. It separates application behavior, release intent, and runtime reconciliation into distinct artifacts, with Git as the sole mechanism for deploy and rollback across dev, staging, and production.

### Repositories and roles

- **[`versioned-app`](https://github.com/dgeoghegan/versioned-app)** — minimal application that reports its running version  
- **[`gitops-release-controller`](https://github.com/dgeoghegan/gitops-release-controller)** — GitOps configuration, environment promotion logic, and primary reviewer artifacts  
- **[`gitops-infra`](https://github.com/dgeoghegan/gitops-infra)** — infrastructure used to host and exercise the demo (not required to understand control flow)

The system is intentionally constrained so change behavior is easy to observe. The **container image tag is the sole mechanism** for changing application runtime state per environment. Deploys and rollbacks are symmetric operations: both are performed by Git PRs that modify versioned configuration, not by imperative cluster actions.

This constraint reduces flexibility and iteration speed. Every change requires a new image build and a Git PR, and ad-hoc hotfixes or partial rollbacks are intentionally disallowed. The tradeoff is that deploys and rollbacks are auditable, reversible, and free of hidden state under incident conditions.

### Fastest way to verify (read-only)

- Open **[`gitops-release-controller/REVIEWER_WALKTHROUGH.md`](https://github.com/dgeoghegan/gitops-release-controller/blob/main/REVIEWER_WALKTHROUGH.md)**
- Within ~60 seconds you will see:
  - a merged PR that changes only `image.tag` in an environment-scoped values file
  - an Argo CD Application manifest pointing at that path
- That single observation validates the core claim:
  - **deploys and rollbacks are controlled solely by Git PRs that modify versioned configuration**

### Optional execution

- Execution is optional and documented in **[`SETUP_GUIDE.md`](https://github.com/dgeoghegan/gitops-release-controller/blob/main/SETUP_GUIDE.md)**
- Credentials are required only if you choose to run the demo end-to-end
- Running the demo shows live reconciliation behavior, timing, and the running application version changing after a PR merge

### Last verified

- **Date:** 2026-02-03  
- **By:** the author  
- **How:** merged PR #24, observed Argo CD reconciliation, confirmed live endpoint returned the expected version

---

## Kubernetes / Terraform Control-Plane  
*(single repository)*

This demo implements a Kubernetes cluster on AWS **using only AWS primitives (EC2, VPC, IAM, and networking)** rather than a managed Kubernetes service such as EKS. Kubernetes control-plane components run directly on EC2 instances provisioned and wired by Terraform, with explicit handling of control-plane configuration, certificates, networking, and availability across multiple availability zones.

The intent is to make the mechanics of the Kubernetes control plane visible rather than collapsed behind a managed service or opinionated platform abstraction.

It focuses on control-plane ownership, lifecycle, and rebuild semantics rather than application-level Kubernetes usage or workload management.

The system is constructed so its behavior can be understood by reading the infrastructure code, **without relying on assumptions or defaults provided by managed Kubernetes platforms**. Dependencies are expressed declaratively, configuration is generated rather than hand-managed, and availability is achieved through explicit placement and wiring rather than implicit defaults. For example, control-plane wiring and certificate handling are modeled directly instead of being assumed by a vendor-managed control plane.

The demo deliberately avoids managed Kubernetes control planes (for example, EKS) and opinionated platform layers, forcing control-plane ordering, dependency, and lifecycle decisions to be made explicit rather than inherited. All supported cluster lifecycle actions are mediated through a single entrypoint script; direct ad-hoc manipulation of infrastructure outside the documented workflow is intentionally unsupported.

**Repository:**  
- **[`kubernetes-cluster-automated`](https://github.com/dgeoghegan/kubernetes-cluster-automated/blob/master/REVIEWER_WALKTHROUGH.md)**

### What “high availability” means here

A single-region, multi-availability-zone cluster with redundant worker capacity and control-plane components distributed across zones. This is not a multi-region or disaster-recovery demo.

### Fastest way to verify (read-only)

- Open **[`kubernetes-cluster-automated/REVIEWER_WALKTHROUGH.md`](https://github.com/dgeoghegan/kubernetes-cluster-automated/blob/master/REVIEWER_WALKTHROUGH.md)**
- Within a few minutes you will see:
  - how the control plane is provisioned on EC2
  - how lifecycle ordering is enforced
  - how rebuilds are supported without workstation state

### Optional execution 

Execution is optional and documented in [`SETUP_GUIDE.md`](https://github.com/dgeoghegan/kubernetes-cluster-automated/blob/master/SETUP_GUIDE.md). Running the demo provisions AWS infrastructure and incurs cloud costs while the environment exists.

Execution requires:
- An AWS account and credentials with permissions to create and destroy VPC networking, EC2 instances, IAM resources, and supporting services.
- Local installation of `terraform` (version ≥ 1.5.0), a Bash shell, `git`, and an SSH client.

All lifecycle actions are performed through a single entrypoint script. A full bring-up provisions a multi–availability zone VPC, EC2-based Kubernetes control-plane and worker nodes, and a dedicated Docker host used to run containerized Ansible and kubectl tooling. Teardown cleanly destroys all provisioned resources when finished.

---

## How the Demos Differ (and Why They’re Separate)

These demos operate at different layers of a Kubernetes-based system.

The control-plane demo focuses on how infrastructure is constructed and reasoned about: configuration, dependencies, and lifecycle expressed so the system can be rebuilt predictably.

The release management demo focuses on how change moves through that infrastructure: how intent is expressed, how responsibility is separated, and how rollback is made routine.

In practice these layers interact, but they are not the same problem. Keeping them separate here keeps each demo small enough to be explainable end to end and prevents their signals from being blurred together.

---

## What This Portfolio Explicitly Does Not Attempt

These demos are intentionally scoped. They are not complete production platforms.

They do not attempt to demonstrate autoscaling strategies, full observability stacks, progressive delivery techniques, multi-region architectures, or organization-specific security hardening. Those concerns are real and important, but they are orthogonal to the questions these demos are designed to answer.

Excluding them keeps the systems small enough to reason about end to end and keeps the focus on explicit configuration, clear boundaries, and verifiable behavior.

---

## Where to Go Next

- **GitOps Release Management**  
  Start with **[`gitops-release-controller/REVIEWER_WALKTHROUGH.md`](https://github.com/dgeoghegan/gitops-release-controller/blob/main/REVIEWER_WALKTHROUGH.md)**. It is the fastest path to evaluating the core claims.

- **Kubernetes / Terraform Control-Plane**  
  Start with **[`kubernetes-cluster-automated`](https://github.com/dgeoghegan/kubernetes-cluster-automated)** and inspect the Terraform root modules to see how control-plane configuration, networking, and availability are modeled explicitly.

Each repository includes its own README and supporting documentation. Execution is optional; the demos are designed so their core claims can be assessed through inspection alone.

---
