# GitHub Actions

## 🎯 Introduction

GitHub Actions is a CI/CD platform integrated into GitHub, enabling automation of workflows directly from your repository.

## 📚 Core Concepts

### Workflow Components
- **Workflows**: Automated processes defined in YAML
- **Events**: Triggers (push, PR, schedule)
- **Jobs**: Sets of steps that run on runners
- **Steps**: Individual tasks
- **Actions**: Reusable workflow components

## 📝 Basic Workflow

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
```

## 🔧 Advanced Features

### Matrix Strategy
```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
    os: [ubuntu-latest, windows-latest]
```

### Secrets
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

### Artifacts
```yaml
- uses: actions/upload-artifact@v3
  with:
    name: build-artifacts
    path: dist/
```

## ✅ Best Practices

- Use specific action versions
- Cache dependencies
- Run tests in parallel
- Use matrix for multiple versions
- Store secrets securely
- Clean up artifacts

---

**Next**: Learn GitLab CI for alternative CI/CD platform.

