# GitHub Actions: Complete Guide to CI/CD Automation

## 🎯 Introduction

GitHub Actions is a powerful CI/CD platform that enables you to automate workflows directly from your GitHub repository. It provides seamless integration with GitHub, extensive marketplace of actions, and flexible workflow definitions using YAML.

### Why GitHub Actions?

**Key Benefits**:
- **Native Integration**: Built into GitHub, no separate service needed
- **Free for Public Repos**: Unlimited minutes for public repositories
- **Extensive Marketplace**: 10,000+ pre-built actions
- **Matrix Builds**: Test across multiple versions simultaneously
- **Built-in Secrets**: Secure secret management
- **Self-hosted Runners**: Run workflows on your own infrastructure
- **Workflow Visualization**: See pipeline status in GitHub UI

### Use Cases

1. **Continuous Integration**: Run tests on every push
2. **Continuous Deployment**: Deploy to staging/production
3. **Code Quality**: Linting, formatting, security scanning
4. **Release Management**: Create releases, publish packages
5. **Automated Tasks**: Issue management, notifications, backups

## 📚 Core Concepts

### Workflow Components

```
Workflow (YAML file)
├── Events (triggers)
├── Jobs (parallel execution units)
│   ├── Runs-on (runner environment)
│   ├── Steps (sequential tasks)
│   │   ├── Actions (reusable components)
│   │   └── Commands (shell commands)
│   └── Strategy (matrix, parallel)
└── Environment (variables, secrets)
```

### Workflow File Location

Workflows are defined in `.github/workflows/` directory:

```
.github/
└── workflows/
    ├── ci.yml          # Continuous Integration
    ├── cd.yml          # Continuous Deployment
    └── release.yml      # Release workflow
```

## 📝 Complete Workflow Examples

### Basic CI Workflow

```yaml
name: Continuous Integration

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Check formatting
        run: npm run format:check

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm test
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella

  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-artifacts
          path: dist/
          retention-days: 7
```

### Complete CI/CD Pipeline

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  release:
    types: [ created ]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: choice
        options:
          - staging
          - production

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Security Scanning
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  # Build and Test
  test:
    name: Test Suite
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        node-version: [18, 20]
        os: [ubuntu-latest, windows-latest]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm test -- --coverage
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-results-${{ matrix.os }}-${{ matrix.node-version }}
          path: test-results/
          retention-days: 30

  # Build Docker Image
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [security, test]
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}
      
      - name: Build and push
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

  # Deploy to Staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'latest'
      
      - name: Setup kustomize
        uses: imranismail/setup-kustomize@v2
        with:
          version: 'latest'
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG_STAGING }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Deploy to Kubernetes
        run: |
          export KUBECONFIG=kubeconfig
          kustomize build k8s/overlays/staging | kubectl apply -f -
          kubectl set image deployment/app app=${{ needs.build.outputs.image-tag }} -n staging
          kubectl rollout status deployment/app -n staging --timeout=5m
      
      - name: Run smoke tests
        run: |
          npm ci
          npm run test:smoke -- --base-url=https://staging.example.com
      
      - name: Notify team
        uses: 8398a7/action-slack@v3
        if: always()
        with:
          status: ${{ job.status }}
          text: 'Deployment to staging ${{ job.status }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}

  # Deploy to Production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/main' || github.event_name == 'release'
    environment:
      name: production
      url: https://example.com
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG_PROD }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Deploy to Kubernetes
        run: |
          export KUBECONFIG=kubeconfig
          kustomize build k8s/overlays/production | kubectl apply -f -
          kubectl set image deployment/app app=${{ needs.build.outputs.image-tag }} -n production
          kubectl rollout status deployment/app -n production --timeout=10m
      
      - name: Run smoke tests
        run: |
          npm ci
          npm run test:smoke -- --base-url=https://example.com
      
      - name: Create GitHub release
        if: github.event_name == 'push'
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ github.run_number }}
          release_name: Release v${{ github.run_number }}
          body: |
            Automated release from commit ${{ github.sha }}
            Image: ${{ needs.build.outputs.image-tag }}
          draft: false
          prerelease: false
```

## 🔧 Advanced Features

### Matrix Strategy

```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
    os: [ubuntu-latest, windows-latest, macos-latest]
    include:
      - node-version: 20
        os: ubuntu-latest
        test-command: 'npm run test:coverage'
    exclude:
      - node-version: 16
        os: windows-latest
  fail-fast: false
  max-parallel: 3
```

### Conditional Execution

```yaml
steps:
  - name: Deploy
    if: github.ref == 'refs/heads/main'
    run: ./deploy.sh
  
  - name: Notify
    if: failure()
    run: ./notify-failure.sh
  
  - name: Cleanup
    if: always()
    run: ./cleanup.sh
```

### Caching

```yaml
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: |
      ~/.npm
      node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- name: Cache Docker layers
  uses: actions/cache@v3
  with:
    path: /tmp/.buildx-cache
    key: ${{ runner.os }}-buildx-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-buildx-
```

### Secrets Management

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
  DATABASE_URL: ${{ secrets.DATABASE_URL }}

steps:
  - name: Use secret
    run: echo "Key is set"
    env:
      SECRET: ${{ secrets.MY_SECRET }}
  
  - name: Mask secret in logs
    run: echo "::add-mask::${{ secrets.SECRET }}"
```

### Artifacts

```yaml
- name: Upload artifacts
  uses: actions/upload-artifact@v3
  with:
    name: build-artifacts
    path: |
      dist/
      coverage/
    retention-days: 30
    if-no-files-found: warn

- name: Download artifacts
  uses: actions/download-artifact@v3
  with:
    name: build-artifacts
    path: ./artifacts
```

### Workflow Dependencies

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
    steps:
      - id: version
        run: echo "::set-output name=version::1.0.0"
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying version ${{ needs.build.outputs.version }}"
```

### Reusable Workflows

**`.github/workflows/reusable-deploy.yml`**:
```yaml
name: Reusable Deploy

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      image-tag:
        required: true
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ${{ inputs.environment }}
        run: |
          echo "Deploying ${{ inputs.image-tag }} to ${{ inputs.environment }}"
```

**Using the workflow**:
```yaml
jobs:
  call-deploy:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: production
      image-tag: v1.0.0
```

## 🎯 Best Practices

### 1. Use Specific Action Versions

```yaml
# Good - Pin to specific version
- uses: actions/checkout@v4

# Bad - Use latest (can break)
- uses: actions/checkout@main
```

### 2. Optimize Workflow Performance

```yaml
# Use caching
- uses: actions/cache@v3
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Run jobs in parallel
jobs:
  test:
  lint:
  build:
  # All run in parallel
```

### 3. Security Best Practices

```yaml
# Use secrets, never hardcode
env:
  API_KEY: ${{ secrets.API_KEY }}

# Use least privilege tokens
permissions:
  contents: read
  packages: write
```

### 4. Error Handling

```yaml
steps:
  - name: Risky operation
    continue-on-error: true
    run: ./risky-script.sh
  
  - name: Check result
    if: failure()
    run: echo "Operation failed but continuing"
```

### 5. Clean Up

```yaml
- name: Cleanup
  if: always()
  run: |
    docker system prune -f
    rm -rf node_modules
```

## 📊 Monitoring and Notifications

### Status Badges

```markdown
![CI](https://github.com/user/repo/workflows/CI/badge.svg)
![CD](https://github.com/user/repo/workflows/CD/badge.svg)
```

### Notifications

```yaml
- name: Notify Slack
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Workflow ${{ github.workflow }} completed'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}

- name: Notify Email
  uses: dawidd6/action-send-mail@v3
  with:
    to: team@example.com
    subject: 'Deployment Status'
    body: 'Deployment ${{ job.status }}'
```

## 🔍 Debugging

### Enable Debug Logging

```yaml
env:
  ACTIONS_STEP_DEBUG: true
  ACTIONS_RUNNER_DEBUG: true
```

### Debug Steps

```yaml
- name: Debug information
  run: |
    echo "::debug::Debug message"
    echo "::warning::Warning message"
    echo "::error::Error message"
    echo "Runner OS: ${{ runner.os }}"
    echo "GitHub SHA: ${{ github.sha }}"
```

## 📚 Common Patterns

### Scheduled Workflows

```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
    - cron: '0 */6 * * *'  # Every 6 hours
    - cron: '0 0 * * 1'   # Every Monday
```

### Manual Triggers

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - staging
          - production
```

### Branch Protection

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

## ✅ Mastery Checklist

- [ ] Create basic CI workflow
- [ ] Use matrix strategy for multiple versions
- [ ] Implement caching for faster builds
- [ ] Set up secrets management
- [ ] Create deployment workflows
- [ ] Use reusable workflows
- [ ] Implement notifications
- [ ] Optimize workflow performance
- [ ] Debug failed workflows
- [ ] Use conditional execution

---

**Next Steps**:
- Learn [GitLab CI](./gitlab-ci.md) for alternative CI/CD
- Explore [Jenkins](./jenkins.md) for self-hosted solutions
- Master [Argo CD](./argo-cd.md) for GitOps deployments

**Remember**: GitHub Actions is powerful and flexible. Start simple, iterate, and continuously optimize your workflows. Use the marketplace for common tasks, and always test workflows thoroughly before deploying to production.
