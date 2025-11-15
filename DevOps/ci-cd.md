# CI/CD: Complete Guide to Continuous Integration and Continuous Deployment

## 🎯 Introduction

Continuous Integration (CI) and Continuous Delivery/Deployment (CD) are fundamental DevOps practices that automate the software delivery process. They enable teams to deliver software faster, more reliably, and with higher quality.

### What is CI/CD?

**Continuous Integration (CI)**:
- Developers frequently integrate code into a shared repository
- Each integration triggers automated builds and tests
- Early detection of integration problems
- Fast feedback on code quality

**Continuous Delivery (CD)**:
- Code is always in a deployable state
- Automated deployment to staging environments
- Manual approval for production deployment
- Low-risk releases

**Continuous Deployment**:
- Every change that passes tests is automatically deployed to production
- No manual intervention required
- Requires high confidence in automated testing
- Fastest feedback loop

### Benefits of CI/CD

1. **Faster Time to Market**: Automated processes reduce manual work
2. **Higher Quality**: Automated testing catches bugs early
3. **Reduced Risk**: Small, frequent changes are less risky
4. **Better Collaboration**: Shared responsibility and visibility
5. **Faster Feedback**: Developers know immediately if something breaks
6. **Easier Rollbacks**: Automated deployments enable quick rollbacks
7. **Consistency**: Same process for all environments
8. **Documentation**: Pipeline serves as executable documentation

## 📚 Core Concepts

### CI/CD Pipeline Stages

```
┌─────────────────────────────────────────────────────────────┐
│                    Source Control                            │
│              (Git Repository: GitHub, GitLab)               │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    Build Stage                               │
│  • Checkout code                                             │
│  • Install dependencies                                       │
│  • Compile/build application                                 │
│  • Create artifacts                                          │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    Test Stage                                │
│  • Unit tests                                                │
│  • Integration tests                                         │
│  • Code quality checks (linting, formatting)                │
│  • Security scanning                                         │
│  • Performance tests                                         │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    Package Stage                            │
│  • Build Docker images                                       │
│  • Package artifacts                                         │
│  • Version tagging                                           │
│  • Push to registry                                          │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    Deploy Stage                              │
│  • Deploy to staging                                         │
│  • Smoke tests                                               │
│  • Deploy to production (manual/automatic)                 │
│  • Health checks                                             │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    Monitor Stage                             │
│  • Monitor application health                                │
│  • Track metrics                                             │
│  • Alert on issues                                           │
│  • Collect feedback                                          │
└─────────────────────────────────────────────────────────────┘
```

### Pipeline Triggers

**Push to Branch**:
- Trigger on every push
- Most common trigger
- Can filter by branch, path, file type

**Pull Request**:
- Trigger on PR creation/update
- Run tests before merge
- Prevent merging if tests fail

**Scheduled**:
- Cron-based triggers
- Nightly builds
- Regular security scans

**Manual**:
- On-demand execution
- Deployment approvals
- Emergency deployments

**Webhooks**:
- External system triggers
- Integration with other tools
- Event-driven pipelines

## 🛠️ CI/CD Tools Comparison

### GitHub Actions

**Pros**:
- Native GitHub integration
- Free for public repositories
- YAML-based configuration
- Large marketplace of actions
- Built-in secrets management
- Matrix builds for multiple versions

**Cons**:
- Limited free minutes for private repos
- GitHub-specific
- Less flexible than self-hosted solutions

**Best For**: GitHub-based projects, open source, small to medium teams

### GitLab CI/CD

**Pros**:
- Integrated with GitLab
- Powerful pipeline features
- Self-hosted option available
- Built-in container registry
- Auto DevOps features
- Multi-project pipelines

**Cons**:
- GitLab-specific
- Can be complex for beginners
- Self-hosted requires maintenance

**Best For**: GitLab users, enterprises, teams wanting integrated solution

### Jenkins

**Pros**:
- Highly customizable
- Extensive plugin ecosystem
- Self-hosted (full control)
- Free and open source
- Supports many languages and tools
- Mature and stable

**Cons**:
- Requires server maintenance
- Can be complex to set up
- Plugin management overhead
- UI can be dated

**Best For**: Large enterprises, complex requirements, full control needed

### CircleCI

**Pros**:
- Cloud-based (no maintenance)
- Fast builds
- Good Docker support
- Free tier available
- Good documentation

**Cons**:
- Cost can scale with usage
- Less customizable than Jenkins
- Vendor lock-in

**Best For**: Teams wanting cloud solution, fast builds, Docker workflows

### Azure DevOps Pipelines

**Pros**:
- Integrated with Azure
- Good Microsoft ecosystem integration
- Free tier for open source
- YAML and UI-based

**Cons**:
- Microsoft ecosystem focused
- Can be complex
- Less popular than alternatives

**Best For**: Azure users, Microsoft ecosystem teams

### AWS CodePipeline

**Pros**:
- Native AWS integration
- Integrates with AWS services
- Pay per use

**Cons**:
- AWS-specific
- Less flexible
- Can be expensive

**Best For**: AWS-native applications, AWS-focused teams

## 📝 Complete Pipeline Examples

### GitHub Actions: Full CI/CD Pipeline

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  release:
    types: [ created ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Lint and Format Check
  lint:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Check formatting
        run: npm run format:check

  # Unit Tests
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20]
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  # Integration Tests
  integration:
    name: Integration Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb

  # Security Scanning
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
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

  # Build Docker Image
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [lint, test, integration]
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    steps:
      - uses: actions/checkout@v4
      
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
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Setup kustomize
        uses: imranismail/setup-kustomize@v2
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Deploy to Kubernetes
        run: |
          export KUBECONFIG=kubeconfig
          kustomize build k8s/overlays/staging | kubectl apply -f -
          kubectl set image deployment/app app=${{ needs.build.outputs.image-tag }}
          kubectl rollout status deployment/app -n staging
      
      - name: Run smoke tests
        run: |
          npm ci
          npm run test:smoke -- --base-url=https://staging.example.com

  # Deploy to Production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/checkout@v4
      
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
          kubectl set image deployment/app app=${{ needs.build.outputs.image-tag }}
          kubectl rollout status deployment/app -n production
      
      - name: Run smoke tests
        run: |
          npm ci
          npm run test:smoke -- --base-url=https://example.com
      
      - name: Notify team
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment to production completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### GitLab CI: Complete Pipeline

```yaml
stages:
  - lint
  - test
  - build
  - security
  - deploy-staging
  - deploy-production

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA

# Lint Stage
lint:code:
  stage: lint
  image: node:20
  script:
    - npm ci
    - npm run lint
    - npm run format:check
  only:
    - merge_requests
    - main
    - develop

# Test Stage
test:unit:
  stage: test
  image: node:20
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

test:integration:
  stage: test
  image: node:20
  services:
    - postgres:15
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
    DATABASE_URL: postgresql://postgres:postgres@postgres:5432/testdb
  script:
    - npm ci
    - npm run test:integration
  only:
    - merge_requests
    - main
    - develop

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
security:scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 0 --severity HIGH,CRITICAL $IMAGE_TAG
    - trivy fs --exit-code 0 --severity HIGH,CRITICAL .
  allow_failure: true
  only:
    - merge_requests
    - main
    - develop

# Deploy to Staging
deploy:staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/app app=$IMAGE_TAG -n staging
    - kubectl rollout status deployment/app -n staging
  only:
    - develop
  when: manual

# Deploy to Production
deploy:production:
  stage: deploy-production
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://example.com
  script:
    - kubectl config use-context production
    - kubectl set image deployment/app app=$IMAGE_TAG -n production
    - kubectl rollout status deployment/app -n production
  only:
    - main
  when: manual
  needs:
    - job: deploy:staging
      artifacts: false
```

### Jenkins: Declarative Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'myapp'
        KUBECONFIG = credentials('kubeconfig')
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
        ansiColor('xterm')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm ci'
                sh 'npm run lint'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm test -- --coverage'
                    }
                    post {
                        always {
                            publishCoverage adapters: [
                                coberturaAdapter('coverage/cobertura-coverage.xml')
                            ]
                        }
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                    def imageLatest = "${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:latest"
                    
                    sh """
                        docker build -t ${imageTag} -t ${imageLatest} .
                        docker push ${imageTag}
                        docker push ${imageLatest}
                    """
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'trivy image ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh """
                    kubectl --kubeconfig=${KUBECONFIG} set image \
                        deployment/app app=${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                        -n staging
                    kubectl --kubeconfig=${KUBECONFIG} rollout status \
                        deployment/app -n staging
                """
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh """
                    kubectl --kubeconfig=${KUBECONFIG} set image \
                        deployment/app app=${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                        -n production
                    kubectl --kubeconfig=${KUBECONFIG} rollout status \
                        deployment/app -n production
                """
            }
        }
    }
    
    post {
        success {
            slackSend(
                color: 'good',
                message: "Build ${env.BUILD_NUMBER} succeeded!"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build ${env.BUILD_NUMBER} failed!"
            )
        }
        always {
            cleanWs()
        }
    }
}
```

## 🎯 Best Practices

### 1. Fast Feedback Loops

**Problem**: Slow pipelines delay feedback
**Solution**:
- Run fast tests first (unit tests)
- Parallelize independent jobs
- Use build caching
- Optimize Docker builds with layer caching
- Use test sharding for large test suites

```yaml
# Example: Parallel test execution
test:
  strategy:
    matrix:
      test-suite: [unit, integration, e2e]
  steps:
    - run: npm run test:${{ matrix.test-suite }}
```

### 2. Fail Fast

**Problem**: Continuing after failures wastes time
**Solution**:
- Stop pipeline on first failure
- Run critical tests first
- Use proper exit codes
- Set timeouts for all jobs

```yaml
jobs:
  test:
    timeout-minutes: 10
    steps:
      - run: npm test || exit 1
```

### 3. Environment Parity

**Problem**: "Works on my machine" syndrome
**Solution**:
- Use containers for consistency
- Match production environment
- Use same OS and versions
- Test with production-like data

### 4. Security First

**Problem**: Vulnerabilities in production
**Solution**:
- Scan dependencies regularly
- Scan Docker images
- Use secrets management
- Implement least privilege
- Regular security audits

```yaml
security:
  steps:
    - name: Dependency scan
      run: npm audit --audit-level=high
    - name: Container scan
      uses: aquasecurity/trivy-action@master
    - name: SAST scan
      uses: github/super-linter@v4
```

### 5. Artifact Management

**Problem**: Rebuilding from scratch wastes time
**Solution**:
- Store build artifacts
- Use artifact repositories
- Version artifacts properly
- Cache dependencies

### 6. Infrastructure as Code

**Problem**: Manual infrastructure changes
**Solution**:
- Version control infrastructure
- Use Terraform/CloudFormation
- Automated infrastructure testing
- Infrastructure in pipeline

### 7. Monitoring and Observability

**Problem**: No visibility into deployments
**Solution**:
- Track deployment metrics
- Monitor application health
- Alert on failures
- Dashboard for visibility

### 8. Rollback Strategy

**Problem**: No way to quickly revert
**Solution**:
- Keep previous versions
- Automated rollback capability
- Blue-green deployments
- Canary deployments

## 🔄 Deployment Strategies

### Rolling Deployment

Gradually replace old instances with new ones.

**Pros**: Zero downtime, gradual rollout
**Cons**: Both versions run simultaneously

```yaml
deploy:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

### Blue-Green Deployment

Run two identical production environments.

**Pros**: Instant rollback, zero downtime
**Cons**: Requires double resources

### Canary Deployment

Gradually route traffic to new version.

**Pros**: Low risk, gradual rollout
**Cons**: Complex setup

### Recreate Deployment

Terminate old version, then deploy new.

**Pros**: Simple, no version conflicts
**Cons**: Downtime during deployment

## 📊 Pipeline Metrics

### Key Metrics to Track

1. **Build Time**: Time to complete pipeline
2. **Build Success Rate**: Percentage of successful builds
3. **Deployment Frequency**: How often you deploy
4. **Lead Time**: Time from commit to production
5. **Mean Time to Recovery (MTTR)**: Time to recover from failures
6. **Change Failure Rate**: Percentage of changes causing failures

### DORA Metrics

- **Elite**: Multiple deploys per day, <1 hour lead time, <1 hour MTTR, 0-15% failure rate
- **High**: Daily to weekly deploys, 1 day-1 week lead time, <1 day MTTR, 16-30% failure rate
- **Medium**: Weekly to monthly deploys, 1-6 months lead time, <1 day MTTR, 31-45% failure rate
- **Low**: Monthly to yearly deploys, 6+ months lead time, >1 day MTTR, 46-60% failure rate

## 🔍 Troubleshooting

### Common Issues

**1. Pipeline Too Slow**
- Use caching
- Parallelize jobs
- Optimize Docker builds
- Use faster runners

**2. Flaky Tests**
- Fix test dependencies
- Add retries for network calls
- Use test containers
- Isolate tests

**3. Build Failures**
- Check logs carefully
- Use verbose output
- Test locally first
- Check dependencies

**4. Deployment Failures**
- Verify environment configuration
- Check resource availability
- Verify secrets
- Test deployment scripts

## ✅ Mastery Checklist

- [ ] Understand CI/CD concepts
- [ ] Set up CI/CD pipeline
- [ ] Implement automated testing
- [ ] Configure deployment automation
- [ ] Set up security scanning
- [ ] Implement monitoring
- [ ] Optimize pipeline performance
- [ ] Handle secrets securely
- [ ] Implement rollback strategy
- [ ] Track pipeline metrics

---

**Next Steps**:
- Learn [GitHub Actions](./github-actions.md) in detail
- Master [GitLab CI](./gitlab-ci.md)
- Explore [Jenkins](./jenkins.md) for complex scenarios
- Understand [Argo CD](./argo-cd.md) for GitOps

**Remember**: CI/CD is about automation and reliability. Start simple, iterate, and continuously improve your pipelines. Measure everything and optimize based on data.
