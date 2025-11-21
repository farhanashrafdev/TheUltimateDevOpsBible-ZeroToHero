# Internal Developer Platform (IDP) with Backstage

## 🎯 Introduction

Backstage (by Spotify) is the open platform for building developer portals. It unifies all your infrastructure tooling, services, and documentation.

## 🧩 Core Features

### 1. Software Catalog

**Single Source of Truth**:
- List all services (APIs, websites, libraries).
- Who owns it?
- Where is the code?
- Is it healthy?

**catalog-info.yaml**:
```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: payment-service
  description: Handles payments
  tags:
    - java
    - payments
spec:
  type: service
  lifecycle: production
  owner: team-payments
  system: payment-system
```

### 2. Software Templates (Scaffolder)

**Golden Paths**:
- Create new projects with best practices built-in.
- "Create Spring Boot Service" -> Generates repo, CI/CD, K8s manifests.

**Template Example**:
```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: spring-boot-template
  title: Spring Boot Service
spec:
  steps:
    - id: fetch-base
      name: Fetch Base
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}
          
    - id: publish
      name: Publish
      action: publish:github
      input:
        allowedHosts: ['github.com']
        description: This is ${{ parameters.name }}
        repoUrl: ${{ parameters.repoUrl }}
```

### 3. TechDocs

**Docs-like-code**:
- Write docs in Markdown next to code.
- Backstage renders them beautifully.

---

## 🛠️ Implementation Guide

### 1. Installation

```bash
npx @backstage/create-app
cd my-backstage-app
yarn dev
```

### 2. Integrations

- **GitHub**: Authentication and reading catalogs.
- **Kubernetes**: View pod status in the portal.
- **Jenkins/CircleCI**: View build status.
- **SonarQube**: View code quality.

### 3. Plugins

Backstage has a huge plugin ecosystem:
- **Cost Insights**: Show cloud spend.
- **Tech Radar**: Standardize technologies.
- **Scorecards**: Gamify service maturity.

---

## ✅ Best Practices

- [ ] Treat the portal as a product (iterate based on feedback)
- [ ] Don't force it; make it so good devs *want* to use it
- [ ] Start with the Catalog (inventory)
- [ ] Add Templates (standardization)
- [ ] Add Docs (knowledge sharing)

---

**Next**: [Golden Paths](./golden-paths.md).
