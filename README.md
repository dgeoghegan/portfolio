# Infrastructure Portfolio — Denis Geoghegan

These projects are presented to illustrate how I approach systems design and implementation. Each is described independently, followed by a set of observations about them as a whole.

---

### 1. Kubernetes Cluster from the Ground Up

**Tooling:** Terraform, Ansible, AWS (EC2)

A full Kubernetes cluster bootstrap inspired by a popular tutorial but restructured using Infrastructure as Code. All cluster identity, networking, and topology are computed from minimal inputs rather than fixed tutorial assumptions, enabling configurable high availability across multiple nodes and availability zones. Execution is containerized with minimal workstation dependencies.

→ [Project Details](https://github.com/dgeoghegan/kubernetes-cluster-automated/blob/master/REVIEWER_WALKTHROUGH.md)

---

### 2. Stripped-down GitOps Control Loop

**Tooling:** GitHub Actions, ArgoCD, Helm, Terraform, EKS

A minimal Git-driven deployment system where application and environment state are fully defined in version control and reconciled continuously. Executes updates and rollbacks of application versions and workloads exclusively through version-controlled commits. Enforces a single declarative control path from repository to cluster state.

→ [Project Details](https://github.com/dgeoghegan/gitops-release-controller/blob/main/REVIEWER_WALKTHROUGH.md)

---

### 3. Image Processing Pipeline in a Weekend

**Tooling:** Python, Gemini API, OpenCV, YOLO

A pipeline that replaces TV screens in a set of images with an alternate picture. (Image files not provided) Combines deterministic computer vision with probabilistic AI fallback mechanisms.

Included to demonstrate how I operate under time constraints in an unfamiliar domain. 

→ [Project Details](https://github.com/dgeoghegan/screen-replacement-pipeline/blob/main/README.md)

---

## Observations

These three projects were built in different contexts and for different purposes, but they share a few structural patterns in how I built them. These are not presented as principles or conclusions, just trends in my work.

- When something failed and I couldn’t see why, the fix was usually making intermediate state visible before execution committed to it. Final output alone was rarely enough to understand what broke.

- Failure modes and recovery paths were designed in from the start, not added after the happy path worked. That shaped both system structure and operational flow.

- Complexity wasn’t eliminated, but was often shifted into forms that improved system legibility. Where my choices made the system less legible, those tradeoffs are documented in the system write-ups.