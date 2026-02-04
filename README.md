# Portfolio Overview — Denis Geoghegan

This page is a guide to a small set of public technical demos. It is not a résumé, a tutorial, or a platform pitch.

These demos are deliberately constrained systems built to surface how infrastructure and delivery problems are decomposed, modeled, and verified under operational constraints. They are **intended to be evaluated through a senior engineering judgment lens**, focusing on tradeoffs, constraints, and falsifiability rather than feature completeness.

These are independent, self-directed demos designed as reviewer-facing artifacts. If this page does its job, you should be able to decide quickly whether you would trust me to reason about infrastructure and delivery systems in production.

These demos are reproducible from scratch and were last verified working in **February 2026**. They are not permanently running services.

---

## How to Read This Portfolio

This page is an orientation layer. You do not need to read everything.

If you only have **five minutes**, start with the **GitOps release management demo** below. It is designed to make its core claim falsifiable quickly, using concrete artifacts intended for read-only inspection.

If you are more interested in Kubernetes internals and infrastructure modeling, read the **control-plane demo** second. It focuses on how a cluster is constructed and rebuilt from explicit configuration.

Both demos are designed so that their core claims *can be evaluated through read-only inspection*. Execution is optional and documented separately for reviewers who want to observe live behavior.

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

### Optional execution (not required for review)

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

This demo implements a Kubernetes cluster on AWS with explicit handling of control-plane configuration, certificates, networking, and availability across multiple availability zones. The intent is to make the mechanics of the control plane visible rather than collapsed behind higher-level abstractions.

The system is constructed so its behavior can be understood by reading the infrastructure code. Dependencies are expressed declaratively, configuration is generated rather than hand-managed, and availability is achieved through explicit placement and wiring rather than implicit defaults. For example, control-plane wiring and certificate handling are modeled directly instead of being assumed by an opinionated platform layer.

The demo avoids higher-level Kubernetes abstractions beyond the managed control plane itself. This forces ordering, dependency, and lifecycle decisions to be made explicitly and makes the consequences of those decisions inspectable.

**Repository:**  
- **[`kubernetes-cluster-automated`](https://github.com/dgeoghegan/kubernetes-cluster-automated)**

### What “high availability” means here

A single-region, multi-availability-zone cluster with redundant worker capacity and control-plane components distributed across zones. This is not a multi-region or disaster-recovery demo.

### Fastest way to verify

- Inspect the Terraform root modules defining control-plane, networking, and AZ layout
- Review generated configuration and certificate handling
- Examine apply and destroy flows demonstrating clean rebuild from a new machine once credentials are supplied

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
