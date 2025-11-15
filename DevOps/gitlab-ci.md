# GitLab CI

## 🎯 Introduction

GitLab CI is a powerful CI/CD platform integrated with GitLab, supporting complex pipelines and self-hosting.

## 📚 Core Concepts

### Pipeline Components
- **Pipelines**: Complete CI/CD process
- **Stages**: Groups of jobs
- **Jobs**: Individual tasks
- **Runners**: Machines that execute jobs

## 📝 Basic Pipeline

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/

test:
  stage: test
  script:
    - npm test

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

## 🔧 Advanced Features

### Multi-stage Pipelines
```yaml
stages:
  - build
  - test
  - deploy-staging
  - deploy-production
```

### Docker Integration
```yaml
build:
  image: node:18
  services:
    - postgres:14
  script:
    - npm test
```

### Conditional Execution
```yaml
deploy:
  script: ./deploy.sh
  only:
    - main
  when: manual
```

## ✅ Best Practices

- Use Docker images
- Cache dependencies
- Parallel jobs
- Environment variables
- Artifacts management
- Security scanning

---

**Next**: Learn Jenkins for self-hosted CI/CD.

