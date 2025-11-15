# Git & GitOps Fundamentals

## 🎯 Introduction

Git is the foundation of modern software development and DevOps. GitOps extends Git principles to infrastructure and operations. This guide covers Git fundamentals and GitOps practices.

## 📚 Git Fundamentals

### Basic Concepts

**Repository**: A directory containing your project and its version history

**Commit**: A snapshot of your project at a point in time

**Branch**: A parallel version of your code

**Remote**: A version of your repository hosted elsewhere (GitHub, GitLab)

### Installation

```bash
# Ubuntu/Debian
sudo apt install git

# RHEL/CentOS
sudo yum install git

# macOS
brew install git

# Verify
git --version
```

### Initial Configuration

```bash
# Set user name
git config --global user.name "Your Name"

# Set email
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set editor
git config --global core.editor "nano"

# View configuration
git config --list
```

## 🔧 Basic Git Commands

### Repository Setup

```bash
# Initialize repository
git init

# Clone repository
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder

# Check status
git status

# View changes
git diff
git diff --staged
```

### Making Changes

```bash
# Add files
git add file.txt              # Add specific file
git add .                     # Add all changes
git add *.txt                 # Add all .txt files
git add -A                    # Add all (including deletions)

# Commit changes
git commit -m "Add feature X"
git commit -am "Quick commit" # Add and commit in one step

# View commit history
git log
git log --oneline
git log --graph --oneline --all
git log -p                    # With diffs
```

### Branching

```bash
# List branches
git branch
git branch -a                 # All branches (including remote)

# Create branch
git branch feature-branch
git checkout -b feature-branch  # Create and switch

# Switch branch
git checkout feature-branch
git switch feature-branch      # Newer command

# Delete branch
git branch -d feature-branch  # Safe delete
git branch -D feature-branch  # Force delete

# Merge branch
git checkout main
git merge feature-branch
```

### Remote Operations

```bash
# View remotes
git remote -v

# Add remote
git remote add origin https://github.com/user/repo.git

# Fetch changes
git fetch origin

# Pull changes
git pull origin main
git pull                      # If tracking branch set

# Push changes
git push origin main
git push                      # If tracking branch set
git push -u origin main       # Set upstream

# Remove remote
git remote remove origin
```

## 🌿 Branching Strategies

### Git Flow

```
main (production)
  │
  ├── develop (integration)
  │     │
  │     ├── feature/user-auth
  │     ├── feature/payment
  │     └── release/1.0.0
  │
  └── hotfix/critical-bug
```

**Workflow**:
- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `release/*`: Release preparation
- `hotfix/*`: Critical fixes

### GitHub Flow (Simpler)

```
main (always deployable)
  │
  ├── feature-branch-1
  ├── feature-branch-2
  └── feature-branch-3
```

**Workflow**:
1. Create feature branch from `main`
2. Make changes and commit
3. Open pull request
4. Review and merge
5. Deploy `main`

### GitLab Flow

Similar to GitHub Flow but with environment branches:
- `main` → `staging` → `production`

## 🔄 Advanced Git

### Stashing

```bash
# Save changes temporarily
git stash
git stash save "Work in progress"

# List stashes
git stash list

# Apply stash
git stash apply
git stash pop                # Apply and remove

# Drop stash
git stash drop stash@{0}
```

### Undoing Changes

```bash
# Unstage file
git reset HEAD file.txt

# Discard changes
git checkout -- file.txt
git restore file.txt         # Newer command

# Amend last commit
git commit --amend -m "New message"

# Reset to previous commit
git reset --soft HEAD~1      # Keep changes staged
git reset --mixed HEAD~1     # Keep changes unstaged
git reset --hard HEAD~1      # Discard all changes (dangerous!)
```

### Rebasing

```bash
# Rebase current branch onto main
git checkout feature-branch
git rebase main

# Interactive rebase
git rebase -i HEAD~3         # Last 3 commits

# Abort rebase
git rebase --abort

# Continue rebase
git rebase --continue
```

### Tags

```bash
# Create tag
git tag v1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"

# List tags
git tag
git tag -l "v1.*"

# Push tags
git push origin v1.0.0
git push --tags              # Push all tags

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0
```

## 🔍 Git Inspection

```bash
# View file history
git log --follow file.txt
git log -p file.txt          # With changes

# Find when bug was introduced
git bisect start
git bisect bad                # Current commit is bad
git bisect good v1.0.0        # This commit was good
# Git will help you find the bad commit

# Search in commits
git log --grep="bug fix"
git log -S "function_name"   # Find when function was added/removed

# View commit details
git show commit-hash
git show HEAD                 # Latest commit
```

## 🤝 Collaboration

### Pull Requests / Merge Requests

**GitHub Pull Request**:
1. Fork or create branch
2. Make changes
3. Push to remote
4. Open pull request
5. Review and discuss
6. Merge when approved

**GitLab Merge Request**: Similar process

### Resolving Conflicts

```bash
# When merge conflict occurs
git merge feature-branch

# Edit conflicted files
# Look for conflict markers:
<<<<<<< HEAD
current code
=======
incoming code
>>>>>>> feature-branch

# After resolving
git add resolved-file.txt
git commit
```

## 🔐 Git Security

### SSH Keys

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Add public key to GitHub/GitLab
cat ~/.ssh/id_ed25519.pub
```

### GPG Signing

```bash
# Generate GPG key
gpg --full-generate-key

# Configure Git
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true

# Sign commits
git commit -S -m "Signed commit"
```

## 🚀 GitOps

### What is GitOps?

GitOps is a methodology where:
- Git is the **single source of truth**
- Infrastructure is **declarative**
- Changes are made via **Git commits**
- Automated systems **sync** Git to infrastructure

### GitOps Principles

1. **Declarative**: Everything described as code
2. **Version Controlled**: All config in Git
3. **Automated**: Automated sync and deployment
4. **Observable**: Monitor and alert on drift

### GitOps Workflow

```
Developer → Git Commit → CI/CD → Git Repository
                                    ↓
                            GitOps Operator
                                    ↓
                            Infrastructure
```

### GitOps Tools

**Argo CD**:
- Declarative GitOps for Kubernetes
- Continuous sync
- Multi-cluster support

**Flux**:
- GitOps operator for Kubernetes
- Automated deployments
- Image automation

**Jenkins X**:
- Cloud-native CI/CD
- GitOps built-in

### Argo CD Example

```yaml
# Application manifest
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### GitOps Best Practices

1. **Separate Repos**: App code vs. infrastructure
2. **Environment Branches**: `dev`, `staging`, `prod`
3. **Automated Sync**: Use operators
4. **Rollback**: Revert Git commit
5. **Audit Trail**: All changes in Git history
6. **Pull Requests**: Review infrastructure changes

## 📝 Git Hooks

### Pre-commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run tests
npm test

# Lint code
npm run lint

# If any fail, prevent commit
if [ $? -ne 0 ]; then
    echo "Tests or linting failed. Commit aborted."
    exit 1
fi
```

### Post-commit Hook

```bash
#!/bin/sh
# .git/hooks/post-commit

# Send notification
echo "Commit $(git rev-parse HEAD) created"
```

## 🎯 Common Workflows

### Feature Development

```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes
git add .
git commit -m "Add new feature"

# 3. Push and create PR
git push -u origin feature/new-feature

# 4. After PR merged, cleanup
git checkout main
git pull
git branch -d feature/new-feature
```

### Hotfix

```bash
# 1. Create hotfix branch from main
git checkout -b hotfix/critical-bug main

# 2. Fix bug
git add .
git commit -m "Fix critical bug"

# 3. Merge to main and develop
git checkout main
git merge hotfix/critical-bug
git checkout develop
git merge hotfix/critical-bug

# 4. Tag release
git tag -a v1.0.1 -m "Hotfix release"
```

## 🔧 Git Aliases

```bash
# Useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## 📊 Git Statistics

```bash
# Commits per author
git shortlog -sn

# Lines changed per author
git log --pretty=tformat: --numstat | \
  awk '{ add += $1; subs += $2; loc += $1 - $2 } END \
  { printf "added lines: %s removed lines: %s total lines: %s\n", add, subs, loc }'

# Files changed
git diff --stat main..feature-branch
```

## ✅ Mastery Checklist

- [ ] Initialize and clone repositories
- [ ] Make commits and view history
- [ ] Create and merge branches
- [ ] Work with remotes
- [ ] Resolve merge conflicts
- [ ] Use stashing and rebasing
- [ ] Create and manage tags
- [ ] Understand GitOps principles
- [ ] Set up GitOps workflows
- [ ] Use Git hooks
- [ ] Collaborate via pull requests
- [ ] Troubleshoot Git issues

---

**Remember**: Git is essential for DevOps. Master the fundamentals, understand branching strategies, and learn GitOps for infrastructure automation. Practice daily!

