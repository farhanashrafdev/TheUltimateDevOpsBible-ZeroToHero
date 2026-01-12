# Developer Portals

## 🎯 Introduction

Developer portals provide a single pane of glass for all developer needs: service catalogs, documentation, APIs, and self-service tooling.

## 📚 Key Components

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Developer Portal Components                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Service Catalog          API Catalog            Documentation      │
│  ├── All services         ├── API registry       ├── TechDocs       │
│  ├── Ownership            ├── OpenAPI specs      ├── Runbooks       │
│  ├── Dependencies         ├── Try it out         ├── Tutorials      │
│  └── Health status        └── Versioning         └── Search         │
│                                                                      │
│  Self-Service             Search                  Metrics           │
│  ├── Create service       ├── Unified search     ├── Build status  │
│  ├── Provision infra      ├── All content        ├── Deploy freq   │
│  └── Request access       └── AI-powered         └── Reliability   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Platform Options

| Platform | Open Source | Managed | Best For |
|----------|-------------|---------|----------|
| Backstage | Yes | Roadie, Spotify | Large orgs, customization |
| Port | No | Yes | Quick setup, integrations |
| Cortex | No | Yes | Scorecards, maturity |
| Compass | No | Atlassian | Jira/Confluence shops |

## 📝 Implementation

### Key Features to Implement

1. **Service Catalog**
   - Auto-discovery from GitHub
   - Ownership assignment
   - Dependency mapping
   - Health indicators

2. **Self-Service Actions**
   - Create new service
   - Provision database
   - Request access
   - Deploy to environment

3. **Documentation**
   - Docs-as-code (Markdown)
   - Searchable
   - Versioned

4. **Integrations**
   - GitHub/GitLab
   - Kubernetes clusters
   - CI/CD systems
   - Monitoring (Datadog, etc.)

## ✅ Success Metrics

- Developer satisfaction (NPS)
- Time to create new service
- Documentation coverage
- Portal adoption rate

---

**Next**: Learn about [Infrastructure Standards](./infra-standards.md).

