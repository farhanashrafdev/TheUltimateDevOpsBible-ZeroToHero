# Backstage: Developer Portal Platform

## 🎯 Introduction

Backstage is Spotify's open-source developer portal platform that provides a unified interface for service catalog, documentation, and self-service infrastructure.

## 📚 Core Features

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Backstage Architecture                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Software Catalog        Software Templates       TechDocs          │
│  ├── Service registry    ├── Project scaffolds   ├── Docs as code  │
│  ├── Ownership           ├── Golden paths        ├── Searchable    │
│  ├── Dependencies        ├── Best practices      ├── Versioned     │
│  └── Health status       └── Self-service        └── Centralized   │
│                                                                      │
│  Kubernetes              Search                   Plugins           │
│  ├── Cluster view        ├── Unified search      ├── 100+ plugins  │
│  ├── Pod status          ├── All content         ├── Custom        │
│  └── Deployments         └── Fast                └── Extensible    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Installation

```bash
# Create new Backstage app
npx @backstage/create-app@latest

# Navigate to app
cd my-backstage-app

# Start development server
yarn dev
```

## 📝 Configuration

### app-config.yaml

```yaml
app:
  title: My Developer Portal
  baseUrl: http://localhost:3000

organization:
  name: My Company

backend:
  baseUrl: http://localhost:7007
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}

integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}

catalog:
  import:
    entityFilename: catalog-info.yaml
  rules:
    - allow: [Component, System, API, Group, User]
  locations:
    - type: url
      target: https://github.com/myorg/*/blob/main/catalog-info.yaml
```

## 📦 Software Catalog

### catalog-info.yaml

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: payment-service
  description: Handles payment processing
  tags:
    - java
    - payments
  annotations:
    github.com/project-slug: myorg/payment-service
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  lifecycle: production
  owner: team-payments
  system: checkout
  providesApis:
    - payment-api
  consumesApis:
    - user-api
  dependsOn:
    - resource:postgres-payments
```

### System Definition

```yaml
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: checkout
  description: Online checkout system
spec:
  owner: team-commerce
  domain: commerce
```

## 🚀 Software Templates

### template.yaml

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: microservice-template
  title: Create a Microservice
  description: Creates a new microservice with CI/CD
spec:
  owner: platform-team
  type: service
  
  parameters:
    - title: Service Info
      required: [name, description]
      properties:
        name:
          title: Name
          type: string
          pattern: '^[a-z0-9-]+$'
        description:
          title: Description
          type: string
        owner:
          title: Owner
          type: string
          ui:field: OwnerPicker
    
    - title: Infrastructure
      properties:
        database:
          title: Database
          type: string
          enum: [postgresql, mysql, none]
          default: postgresql
  
  steps:
    - id: fetch-base
      name: Fetch Template
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}
          description: ${{ parameters.description }}
    
    - id: publish
      name: Publish to GitHub
      action: publish:github
      input:
        repoUrl: github.com?owner=myorg&repo=${{ parameters.name }}
        description: ${{ parameters.description }}
    
    - id: register
      name: Register in Catalog
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: /catalog-info.yaml
```

## 📚 TechDocs

### mkdocs.yml

```yaml
site_name: Payment Service
nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - API Reference: api.md
  - Runbooks:
      - Deployment: runbooks/deployment.md
      - Troubleshooting: runbooks/troubleshooting.md

plugins:
  - techdocs-core
```

### Enable in app-config.yaml

```yaml
techdocs:
  builder: 'local'
  publisher:
    type: 'local'
  generator:
    runIn: 'local'
```

## 🔌 Kubernetes Plugin

```yaml
# app-config.yaml
kubernetes:
  serviceLocatorMethod:
    type: 'multiTenant'
  clusterLocatorMethods:
    - type: 'config'
      clusters:
        - url: ${K8S_CLUSTER_URL}
          name: production
          authProvider: 'serviceAccount'
          serviceAccountToken: ${K8S_TOKEN}
```

## ✅ Best Practices

1. **Start with Catalog**: Register all services first
2. **Define Ownership**: Clear team ownership for all components
3. **Document Everything**: TechDocs for all services
4. **Create Templates**: Golden paths for new services
5. **Enable Plugins**: GitHub, Kubernetes, CI/CD integrations

---

**Next**: Learn about [Golden Paths](./golden-paths.md).

