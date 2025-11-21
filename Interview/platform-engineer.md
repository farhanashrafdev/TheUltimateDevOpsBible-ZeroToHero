# Platform Engineer Interview Questions

## 🎯 Introduction
Platform Engineering focuses on building the "Internal Developer Platform" (IDP) to enable self-service.

## 🟢 Philosophy & Culture

### 1. Why Platform Engineering?
**Answer**: To reduce cognitive load on developers. Instead of every dev learning K8s/Terraform, the platform provides "Golden Paths" (paved roads) for common tasks. It treats the platform as a product and developers as customers.

### 2. What is a "Golden Path"?
**Answer**: An opinionated, supported, and automated way to build and deploy software. E.g., "Spring Boot on K8s" template that comes with CI/CD, monitoring, and security pre-configured.

### 3. How do you measure the success of a Platform?
**Answer**:
- **Adoption Rate**: % of teams using the platform.
- **Time to First Hello World**: How fast can a new dev ship code?
- **DORA Metrics**: Does the platform improve deployment frequency?
- **NPS**: Developer satisfaction surveys.

---

## 🟡 Tooling & Architecture

### 4. Explain the concept of "Internal Developer Portal".
**Answer**: A UI (like Backstage) that serves as a single pane of glass.
- **Service Catalog**: Inventory of all software.
- **Scaffolder**: Create new services from templates.
- **Docs**: TechDocs.
- **Scorecards**: Measure service maturity.

### 5. Kubernetes: What is a Custom Resource Definition (CRD)?
**Answer**: Extends the K8s API. Allows you to define your own resources (e.g., `Database`, `S3Bucket`) that act like native K8s objects. Used heavily in Operators and Crossplane.

### 6. Compare Terraform vs. Crossplane.
**Answer**:
- **Terraform**: CLI-based, state file, runs in CI/CD. Good for "static" infra.
- **Crossplane**: K8s-based, control plane, continuous reconciliation. Good for "dynamic" self-service infra and preventing drift.

### 7. How do you handle "Configuration Drift"?
**Answer**:
- **Terraform**: Run `terraform plan` regularly (GitOps).
- **Crossplane**: Automatically corrects drift (Control Loop).
- **Policy**: OPA/Kyverno to block manual changes.

---

## 🔴 Advanced Scenarios

### 8. Design a self-service database provisioning system.
**Answer**:
1. **Interface**: Dev clicks "Create DB" in Backstage.
2. **Trigger**: Backstage commits a YAML file to a Git repo (GitOps).
3. **Orchestrator**: Argo CD syncs the YAML to Kubernetes.
4. **Provisioner**: Crossplane (or Terraform Operator) sees the CRD and calls AWS API to create RDS.
5. **Output**: Connection string written to a Kubernetes Secret.
6. **Access**: App reads Secret.

### 9. How do you upgrade a Kubernetes cluster with zero downtime?
**Answer**:
- **Blue/Green Cluster**: Spin up new cluster, switch traffic. (Expensive).
- **Rolling Node Upgrade**: Cordon node -> Drain node (pods move) -> Terminate node -> Launch new node.
- **Control Plane**: Cloud provider handles this (EKS/GKE).

### 10. Developers complain CI is slow. How do you fix it?
**Answer**:
- **Cache**: Cache dependencies (node_modules, pip).
- **Docker**: Use multi-stage builds, cache layers.
- **Parallelism**: Run tests in parallel.
- **Ephemeral Agents**: Use spot instances for build agents.
- **Bazel/Nx**: Use build systems that support incremental builds.

---

## 🧠 Coding (Go/Python)

### 11. Write a Kubernetes Operator (Conceptual).
**Answer**:
1. Define CRD (`MyService`).
2. Write Controller loop (Reconcile function).
3. Watch for changes to `MyService`.
4. Compare `Desired State` (Spec) vs `Actual State`.
5. Apply changes to match Desired State.

### 12. Parse a YAML file and validate fields.
**Answer**: Use `pyyaml` or `encoding/json` (Go) to unmarshal and check required fields.
