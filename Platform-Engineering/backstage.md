# Backstage

## 🎯 Introduction

Backstage is an open-source platform for building developer portals, providing service catalog, software templates, and tech docs.

## 📚 Core Features

### Service Catalog
- Centralized service registry
- Service ownership
- Health status
- Documentation

### Software Templates
- Scaffold new services
- Standardized patterns
- Custom templates

### Tech Docs
- Documentation as code
- Version control
- Searchable

## 📝 Basic Setup

```yaml
# app-config.yaml
app:
  title: My Backstage
  baseUrl: http://localhost:7007

backend:
  baseUrl: http://localhost:7007
  database:
    client: better-sqlite3
    connection: ':memory:'

integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}
```

## 🔧 Key Concepts

### Components
- Services, websites, libraries
- Defined in YAML or code

### Templates
- Scaffold new projects
- Customizable workflows

### Plugins
- Extend functionality
- Custom integrations

## ✅ Best Practices

- Start with service catalog
- Create software templates
- Integrate with CI/CD
- Document services
- Enable self-service

---

**Next**: Learn about golden paths.

