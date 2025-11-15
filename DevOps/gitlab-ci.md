# GitLab CI/CD: Complete Guide to Pipeline Automation

## 🎯 Introduction

GitLab CI/CD is a powerful continuous integration and deployment platform integrated directly into GitLab. It enables you to build, test, and deploy applications using a single `.gitlab-ci.yml` file in your repository.

### Why GitLab CI/CD?

**Key Benefits**:
- **Integrated Platform**: CI/CD, version control, and issue tracking in one place
- **Powerful Pipelines**: Complex multi-stage pipelines with dependencies
- **Self-Hosted Option**: Run on your own infrastructure
- **Built-in Container Registry**: Store Docker images directly in GitLab
- **Auto DevOps**: Pre-configured CI/CD templates
- **Multi-Project Pipelines**: Coordinate deployments across projects
- **Security Scanning**: Built-in SAST, DAST, dependency scanning

### Use Cases

1. **Continuous Integration**: Automated testing on every commit
2. **Continuous Deployment**: Deploy to multiple environments
3. **Container Builds**: Build and push Docker images
4. **Security Scanning**: Automated vulnerability detection
5. **Performance Testing**: Load and performance tests
6. **Infrastructure as Code**: Deploy infrastructure changes

## 📚 Core Concepts

### Pipeline Components

```
Pipeline
├── Stages (sequential groups)
│   ├── Stage 1: build
│   │   ├── Job 1: build-frontend
│   │   └── Job 2: build-backend
│   ├── Stage 2: test
│   │   ├── Job 1: unit-tests
│   │   ├── Job 2: integration-tests
│   │   └── Job 3: e2e-tests
│   └── Stage 3: deploy
│       └── Job 1: deploy-production
└── Variables (shared configuration)
```

### GitLab CI/CD File

The `.gitlab-ci.yml` file defines your pipeline:

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_DRIVER: overlay2

build:
  stage: build
  script:
    - echo "Building application"
```

## 📝 Complete Pipeline Examples

### Basic CI Pipeline

```yaml
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "20"
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

# Build Stage
build:frontend:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  only:
    - merge_requests
    - main
    - develop

build:backend:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run build:server
  artifacts:
    paths:
      - server-dist/
    expire_in: 1 week

# Test Stage
test:unit:
  stage: test
  image: node:${NODE_VERSION}
  dependencies:
    - build:frontend
  script:
    - npm ci
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 1 week
  only:
    - merge_requests
    - main
    - develop

test:integration:
  stage: test
  image: node:${NODE_VERSION}
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
    DATABASE_URL: postgresql://postgres:postgres@postgres:5432/testdb
    REDIS_URL: redis://redis:6379
  script:
    - npm ci
    - npm run test:integration
  only:
    - merge_requests
    - main

test:e2e:
  stage: test
  image: cypress/included:latest
  script:
    - npm ci
    - npm run test:e2e
  artifacts:
    when: always
    paths:
      - cypress/screenshots/
      - cypress/videos/
    expire_in: 1 week
  only:
    - main

# Deploy Stage
deploy:staging:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA -n staging
    - kubectl rollout status deployment/app -n staging
  only:
    - develop
  when: manual

deploy:production:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://example.com
  script:
    - kubectl config use-context production
    - kubectl set image deployment/app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA -n production
    - kubectl rollout status deployment/app -n production
  only:
    - main
  when: manual
  needs:
    - job: deploy:staging
      artifacts: false
```

### Advanced Multi-Stage Pipeline

```yaml
stages:
  - validate
  - build
  - test
  - security
  - package
  - deploy-staging
  - deploy-production

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA

# Validation Stage
validate:yaml:
  stage: validate
  image: alpine:latest
  script:
    - apk add --no-cache yamllint
    - yamllint .gitlab-ci.yml
    - find . -name "*.yaml" -o -name "*.yml" | xargs yamllint

validate:terraform:
  stage: validate
  image: hashicorp/terraform:latest
  script:
    - terraform init -backend=false
    - terraform validate
    - terraform fmt -check
  only:
    changes:
      - terraform/**/*

# Build Stage
build:docker:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG -t $CI_REGISTRY_IMAGE:latest .
    - docker push $IMAGE_TAG
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop
    - tags

# Security Stage
security:sast:
  stage: security
  image: node:20
  script:
    - npm audit --audit-level=high
  allow_failure: true
  only:
    - merge_requests
    - main

security:container-scan:
  stage: security
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy image --exit-code 0 --severity HIGH,CRITICAL $IMAGE_TAG
  allow_failure: true
  only:
    - main
    - develop

security:iac-scan:
  stage: security
  image:
    name: bridgecrew/checkov:latest
    entrypoint: [""]
  script:
    - checkov -d terraform/ --framework terraform
  allow_failure: true
  only:
    changes:
      - terraform/**/*

# Package Stage
package:helm:
  stage: package
  image: alpine/helm:latest
  script:
    - helm package ./charts/myapp
    - helm push myapp-*.tgz oci://$CI_REGISTRY/helm
  artifacts:
    paths:
      - *.tgz
    expire_in: 1 week
  only:
    - tags

# Deploy Stages
deploy:staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop-staging
  script:
    - kubectl config use-context staging
    - helm upgrade --install myapp ./charts/myapp \
        --namespace staging \
        --set image.tag=$CI_COMMIT_SHORT_SHA \
        --wait
  only:
    - develop
  when: manual

stop-staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  environment:
    name: staging
    action: stop
  script:
    - kubectl config use-context staging
    - helm uninstall myapp --namespace staging
  when: manual
  only:
    - develop

deploy:production:
  stage: deploy-production
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://example.com
  script:
    - kubectl config use-context production
    - helm upgrade --install myapp ./charts/myapp \
        --namespace production \
        --set image.tag=$CI_COMMIT_SHORT_SHA \
        --wait \
        --timeout 10m
  only:
    - main
  when: manual
  needs:
    - job: deploy:staging
      artifacts: false
```

## 🔧 Advanced Features

### Parallel Jobs

```yaml
test:unit:
  stage: test
  parallel:
    matrix:
      - NODE_VERSION: ["18", "20"]
        OS: ["ubuntu-latest", "alpine"]
  image: node:${NODE_VERSION}-${OS}
  script:
    - npm test
```

### Job Dependencies

```yaml
deploy:
  stage: deploy
  needs:
    - job: build
      artifacts: true
    - job: test
      artifacts: false
  script:
    - ./deploy.sh
```

### Rules (Advanced Conditions)

```yaml
deploy:
  script: ./deploy.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: never
    - if: $CI_COMMIT_TAG
      when: on_success
    - when: manual
```

### Cache

```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .cache/
  policy: pull-push

# Or per-job cache
test:
  cache:
    key: ${CI_COMMIT_REF_SLUG}-npm
    paths:
      - node_modules/
    policy: pull
```

### Artifacts

```yaml
build:
  artifacts:
    paths:
      - dist/
      - coverage/
    reports:
      junit: test-results.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    expire_in: 1 week
    when: on_success
```

### Services

```yaml
test:
  services:
    - name: postgres:15
      alias: postgres
    - name: redis:7
      alias: redis
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
    DATABASE_URL: postgresql://postgres:postgres@postgres:5432/testdb
```

### Multi-Project Pipelines

```yaml
# Project A
trigger:project-b:
  stage: deploy
  trigger:
    project: group/project-b
    branch: main
    strategy: depend

# Project B (triggered)
build:
  stage: build
  script:
    - echo "Building in project B"
```

## 🎯 Best Practices

### 1. Use Docker Images

```yaml
# Good - Specific version
image: node:20-alpine

# Bad - Latest (unpredictable)
image: node:latest
```

### 2. Optimize Caching

```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
  policy: pull-push
```

### 3. Use Artifacts Efficiently

```yaml
artifacts:
  paths:
    - dist/  # Only necessary files
  expire_in: 1 week  # Don't keep forever
```

### 4. Security

```yaml
# Use CI/CD variables for secrets
variables:
  DATABASE_URL: $DATABASE_URL_SECRET

# Don't hardcode secrets
# Bad: DATABASE_URL: "postgresql://user:password@host:5432/db"
```

### 5. Parallel Execution

```yaml
# Run independent jobs in parallel
test:unit:
test:integration:
test:e2e:
# All run simultaneously
```

## 📊 GitLab CI/CD Variables

### Predefined Variables

```yaml
# Common variables
$CI_COMMIT_SHA              # Commit SHA
$CI_COMMIT_REF_NAME         # Branch or tag name
$CI_COMMIT_REF_SLUG         # Branch/tag slug
$CI_PROJECT_NAME            # Project name
$CI_PROJECT_PATH            # Project path
$CI_JOB_NAME                # Job name
$CI_PIPELINE_ID             # Pipeline ID
$CI_REGISTRY                # Container registry URL
$CI_REGISTRY_IMAGE          # Container registry image
$CI_PIPELINE_SOURCE         # Pipeline source (push, web, etc.)
```

### Custom Variables

Set in GitLab UI: Settings → CI/CD → Variables

```yaml
variables:
  CUSTOM_VAR: "value"
  SECRET_VAR: $SECRET_VAR  # From GitLab CI/CD variables
```

## 🔍 Debugging

### Enable Debug Mode

```yaml
variables:
  CI_DEBUG_TRACE: "true"
```

### Debug Commands

```yaml
debug:
  script:
    - echo "Debug info"
    - env | grep CI_
    - kubectl version --client
    - docker info
```

## 📚 Common Patterns

### Conditional Deployment

```yaml
deploy:
  script: ./deploy.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
    - when: never
```

### Scheduled Pipelines

```yaml
# In GitLab UI: CI/CD → Schedules
# Or use API
```

### Manual Jobs

```yaml
deploy:production:
  when: manual
  script: ./deploy.sh
```

## ✅ Mastery Checklist

- [ ] Create basic pipeline
- [ ] Use multiple stages
- [ ] Implement caching
- [ ] Use artifacts
- [ ] Set up Docker builds
- [ ] Configure environments
- [ ] Use rules for conditional execution
- [ ] Implement security scanning
- [ ] Set up multi-project pipelines
- [ ] Optimize pipeline performance

---

**Next Steps**:
- Learn [Jenkins](./jenkins.md) for self-hosted CI/CD
- Explore [GitHub Actions](./github-actions.md) for GitHub integration
- Master [Argo CD](./argo-cd.md) for GitOps

**Remember**: GitLab CI/CD is powerful and flexible. Start with simple pipelines and gradually add complexity. Use the built-in templates and examples, and always test your pipelines thoroughly.
