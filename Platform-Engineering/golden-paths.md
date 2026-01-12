# Golden Paths

## 🎯 Introduction

Golden Paths are the recommended, supported ways to accomplish common tasks within an organization. They provide standardized, well-tested approaches that reduce friction and increase consistency.

## 📚 What are Golden Paths?

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Golden Paths                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Without Golden Paths:           With Golden Paths:                 │
│                                                                      │
│  Team A: Express + MySQL         All Teams:                         │
│  Team B: Flask + PostgreSQL      ├── Service template with CI/CD   │
│  Team C: FastAPI + MongoDB       ├── Standard observability        │
│  Team D: Custom everything       ├── Security best practices       │
│                                   ├── Kubernetes deployment        │
│  = Inconsistency, tech debt      └── Documentation template        │
│  = Hard to support                                                   │
│  = Security gaps                 = Consistency, speed               │
│                                   = Easy to support                  │
│                                   = Security by default             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Examples

### Service Creation Golden Path

```yaml
# Backstage Software Template
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: golden-service
  title: Standard Microservice
spec:
  parameters:
    - title: Service Details
      properties:
        name:
          type: string
        language:
          type: string
          enum: [go, python, nodejs]
        team:
          type: string
          ui:field: OwnerPicker
  
  steps:
    # Scaffold with standards
    - id: template
      action: fetch:template
      input:
        url: ./template-${{ parameters.language }}
    
    # Create repo with branch protection
    - id: publish
      action: publish:github
      input:
        repoUrl: github.com?owner=org&repo=${{ parameters.name }}
        branchProtectionEnabled: true
    
    # Setup CI/CD
    - id: cicd
      action: github:actions:dispatch
      input:
        workflow: setup-cicd.yml
    
    # Register in catalog
    - id: register
      action: catalog:register
```

### What's Included

```
golden-service/
├── .github/
│   └── workflows/
│       ├── ci.yml           # Standard CI pipeline
│       ├── cd.yml           # Standard CD pipeline
│       └── security.yml     # Security scanning
├── kubernetes/
│   ├── deployment.yaml      # K8s manifests
│   ├── service.yaml
│   └── networkpolicy.yaml   # Security by default
├── src/                     # Application code
├── tests/                   # Test structure
├── docs/                    # Documentation
├── Dockerfile               # Optimized, secure
├── catalog-info.yaml        # Backstage registration
└── README.md                # Getting started guide
```

### CI/CD Golden Path

```yaml
# .github/workflows/ci.yml (included in template)
name: CI Pipeline

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make test
      - uses: codecov/codecov-action@v3

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'

  build:
    needs: [lint, test, security]
    runs-on: ubuntu-latest
    steps:
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/org/${{ github.repository }}:${{ github.sha }}
```

## 📊 Measuring Adoption

Track golden path usage:
- % of new services using templates
- Time to first deployment
- Security scan pass rates
- Developer satisfaction

## ✅ Best Practices

1. **Start Simple**: One golden path per use case
2. **Make it Easy**: Easier than DIY alternatives
3. **Keep Updated**: Regular maintenance and improvements
4. **Get Feedback**: Iterate based on developer input
5. **Don't Force**: Encourage, don't mandate

---

**Next**: Learn about [Developer Portals](./developer-portals.md).

