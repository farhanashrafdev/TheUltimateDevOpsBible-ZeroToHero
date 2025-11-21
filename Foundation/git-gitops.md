# Git & GitOps: Version Control to Infrastructure Automation

> **"Git is the undo button for your career."**

## 🎯 Introduction

Git is more than just `commit` and `push`. It's the foundation of modern DevOps. **GitOps** takes this further: your entire infrastructure is defined in Git, and automated systems sync Git to reality.

This guide covers **practical Git workflows and GitOps patterns for production systems**.

### What You'll Learn
- Git workflows for teams
- Advanced Git techniques (reflog, worktree, cherry-pick)
- GitOps principles and architecture
- Argo CD vs Flux
- Secrets management in GitOps

### What This Is NOT
- Not a Git basics tutorial (`git init`, `git add`)
- Not every Git command
- Not Git internals (plumbing commands)

---

## 🌿 Git Workflows for Teams

### 1. Trunk-Based Development (Recommended for DevOps)

**Philosophy**: Everyone pushes to `main` (or short-lived feature branches < 1 day).

```
main ─────●─────●─────●─────●─────●
           \   /       \   /
            ●─●         ●─●
         (feature)   (feature)
```

**Benefits**:
- Fast feedback (CI runs on every commit)
- No merge hell
- Forces small, incremental changes

**How to do it**:
- Feature flags for incomplete features
- Automated testing catches issues early
- Deploy `main` frequently

**Real-world example (Google)**:
- 25,000+ engineers commit to one monorepo
- Trunk-based development
- Automated testing at massive scale

### 2. GitFlow (Legacy/Enterprise)

**Philosophy**: Complex branching model with `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`.

```
main ─────●───────────●───────────●
           \           \           \
develop ────●─────●─────●─────●─────●
             \   /       \   /
              ●─●         ●─●
           (feature)   (feature)
```

**When to use**: Boxed software with scheduled releases

**When NOT to use**: Continuous delivery (it's too slow)

---

## 🔧 Advanced Git Techniques

### 1. The Lifesaver: `git reflog`

**Deleted a branch? Reset to the wrong commit?** `reflog` saves you.

```bash
# View history of HEAD movements
git reflog

# Output:
# abc1234 HEAD@{0}: commit: Add feature
# def5678 HEAD@{1}: reset: moving to HEAD~1
# ghi9012 HEAD@{2}: commit: Oops, deleted this

# Recover lost commit
git reset --hard HEAD@{2}
```

**Real-world scenario**:
```bash
# Accidentally deleted branch
git branch -D feature-branch

# Recover it
git reflog | grep feature-branch
git checkout -b feature-branch abc1234
```

### 2. Parallel Work: `git worktree`

Need to fix a bug on `main` while working on `feature-x`? Don't stash. Use worktrees.

```bash
# Create a new folder linked to 'main' branch
git worktree add ../hotfix-folder main

# Do your work in ../hotfix-folder
cd ../hotfix-folder
# ... fix bug, commit, push ...

# Remove worktree
cd ../original-repo
git worktree remove ../hotfix-folder
```

### 3. Surgical Precision: `git cherry-pick`

Apply a specific commit from one branch to another.

```bash
# Apply commit abc1234 to current branch
git cherry-pick abc1234

# Cherry-pick multiple commits
git cherry-pick abc1234 def5678

# Cherry-pick without committing (review first)
git cherry-pick -n abc1234
```

**Real-world use case**:
```bash
# Hotfix was committed to 'develop', need it in 'main'
git checkout main
git cherry-pick hotfix-commit-hash
```

### 4. Finding Bugs: `git bisect`

Automated binary search to find the commit that introduced a bug.

```bash
# Start bisect
git bisect start

# Current version is broken
git bisect bad

# v1.0 was working
git bisect good v1.0

# Git checks out middle commit
# Test it, then mark as good or bad
git bisect good   # or git bisect bad

# Repeat until Git finds the bad commit
# When done:
git bisect reset
```

---

## 🚀 GitOps: Infrastructure as Code V2

**GitOps** is Continuous Delivery for Cloud Native applications.

### The 4 Principles of GitOps

1. **Declarative**: The entire system described as code (YAML/HCL)
2. **Versioned**: The canonical desired state is in Git
3. **Automated**: Changes are automatically applied
4. **Self-Healing**: Software agents ensure current state matches desired state

### Push vs. Pull Model

#### ➡️ Push Model (Classic CI/CD)

Jenkins/GitLab CI executes `kubectl apply`.

```
Developer → Git Push → CI/CD → kubectl apply → Kubernetes
```

**Pros**:
- Simple to set up
- Familiar pattern

**Cons**:
- CI needs admin access to cluster (security risk)
- No drift detection
- Manual rollback

#### ⬅️ Pull Model (True GitOps)

An agent (ArgoCD/Flux) inside the cluster pulls changes from Git.

```
Developer → Git Push → Git Repo
                          ↓
                    GitOps Operator (in cluster)
                          ↓
                      Kubernetes
```

**Pros**:
- Cluster doesn't need external credentials
- Detects and fixes drift automatically
- Git is the source of truth

**Cons**:
- Requires running an operator

---

## 🛠️ GitOps Tools: Argo CD vs Flux

| Feature | Argo CD | Flux |
|:---|:---|:---|
| **UI** | Excellent web UI | CLI-focused (UI is separate) |
| **Architecture** | Centralized (Hub & Spoke) | Decentralized (Controller per cluster) |
| **Multi-cluster** | Native support | Requires setup |
| **Helm Support** | Excellent | Excellent |
| **Kustomize Support** | Excellent | Excellent |
| **Best For** | App Developers & Platform Teams | Kubernetes Admins & Infrastructure |

### Argo CD Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    automated:
      prune: true        # Delete resources not in Git
      selfHeal: true     # Revert manual changes
```

**Real-world workflow**:
1. Developer pushes to Git
2. Argo CD detects change
3. Argo CD syncs to Kubernetes
4. If someone manually edits a deployment, Argo CD reverts it (self-heal)

### Flux Example

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/myorg/my-app
  ref:
    branch: main
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 5m
  path: ./kustomize
  prune: true
  sourceRef:
    kind: GitRepository
    name: my-app
```

---

## 🔐 Secrets in GitOps

**NEVER commit raw secrets to Git.**

### 1. Sealed Secrets (Bitnami)

Encrypt secrets locally, commit encrypted version to Git.

```bash
# Install kubeseal CLI
brew install kubeseal

# Create secret
kubectl create secret generic mysecret --dry-run=client -o yaml \
  --from-literal=password=supersecret > mysecret.yaml

# Seal it (encrypt)
kubeseal -f mysecret.yaml -w mysealedsecret.yaml

# Commit mysealedsecret.yaml to Git
git add mysealedsecret.yaml
git commit -m "Add sealed secret"

# Controller in cluster decrypts it
kubectl apply -f mysealedsecret.yaml
```

**How it works**:
1. Sealed Secrets controller generates a key pair
2. You encrypt with public key
3. Controller decrypts with private key

### 2. External Secrets Operator (ESO)

Store secrets in AWS Secrets Manager / HashiCorp Vault, ESO fetches them.

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: my-k8s-secret
  data:
    - secretKey: password
      remoteRef:
        key: prod/db/password
```

**Benefits**:
- Centralized secret management
- Audit trail
- Rotation support

### 3. SOPS (Mozilla)

Encrypt YAML files using AWS KMS / PGP.

```bash
# Install sops
brew install sops

# Encrypt file
sops -e secrets.yaml > secrets.enc.yaml

# Commit encrypted file
git add secrets.enc.yaml

# Decrypt (Flux does this automatically)
sops -d secrets.enc.yaml
```

---

## 🎯 GitOps Best Practices

### 1. Separate Repos

**App Code** vs **Infrastructure Code**

```
myapp-code/          # Application source code
├── src/
├── Dockerfile
└── .github/workflows/

myapp-infra/         # Kubernetes manifests
├── base/
├── overlays/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── argocd/
```

**Why?**
- Different teams (app devs vs platform team)
- Different change frequency
- Different access controls

### 2. Environment Branches vs Directories

**Option A: Branches**
```
main → production
staging → staging environment
dev → dev environment
```

**Option B: Directories (Recommended)**
```
overlays/
├── dev/
├── staging/
└── prod/
```

**Why directories?**
- Easier to see differences
- No merge conflicts between environments
- Works better with Kustomize

### 3. Automated Sync with Manual Approval for Prod

```yaml
# Dev: Auto-sync
syncPolicy:
  automated:
    prune: true
    selfHeal: true

# Prod: Manual approval
syncPolicy:
  automated: false  # Require manual sync
```

---

## ✅ Mastery Checklist

After mastering this guide, you should be able to:

- [ ] Recover lost commits with `git reflog`
- [ ] Use `git worktree` for parallel work
- [ ] Cherry-pick commits between branches
- [ ] Explain Trunk-Based Development
- [ ] Understand GitOps principles (Declarative, Versioned, Automated, Self-Healing)
- [ ] Explain Push vs Pull model
- [ ] Set up Argo CD or Flux
- [ ] Manage secrets securely (Sealed Secrets, ESO, SOPS)
- [ ] Implement a GitOps workflow for production

---

## 📚 Practice Exercises

### Exercise 1: Recover Lost Work
```bash
# Create a commit
echo "test" > file.txt
git add file.txt
git commit -m "Test commit"

# Accidentally reset
git reset --hard HEAD~1

# Recover using reflog
git reflog
git reset --hard HEAD@{1}
```

### Exercise 2: GitOps Setup
1. Install Argo CD in a local Kubernetes cluster
2. Create a Git repo with a simple app manifest
3. Configure Argo CD to sync from your repo
4. Make a change in Git, watch Argo CD sync it

### Exercise 3: Sealed Secrets
1. Install Sealed Secrets controller
2. Create a secret
3. Seal it
4. Commit to Git
5. Verify it's decrypted in the cluster

---

**Next Step**: Now that we can manage code and infrastructure, let's automate everything. Master **[Scripting with Bash & Python](./scripting-bash-python.md)**.

> **Remember**: Git is your safety net. Commit often. Push regularly. GitOps makes Git the source of truth for your entire infrastructure. If it's not in Git, it doesn't exist.
