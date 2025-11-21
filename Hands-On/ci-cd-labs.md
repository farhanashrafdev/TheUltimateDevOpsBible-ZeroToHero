# CI/CD Hands-On Labs: Complete Practical Guide

## 🎯 Introduction

This comprehensive guide provides hands-on labs to master CI/CD pipelines. Each lab builds practical skills with real-world scenarios.

## 📚 Lab Structure

Each lab includes:
- **Objective**: Learning goals
- **Prerequisites**: Required setup
- **Step-by-Step Tasks**: Detailed instructions
- **Code Examples**: Complete configurations
- **Verification**: Success criteria
- **Troubleshooting**: Common issues
- **Challenges**: Advanced exercises

## 🔄 Lab 1: GitHub Actions CI Pipeline

### Objective
Build a complete CI pipeline with GitHub Actions for a Node.js application.

### Prerequisites
- GitHub account
- Basic Git knowledge
- Node.js application (or create one)

### Tasks

#### Task 1: Create Basic CI Pipeline

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Generate coverage report
        run: npm run test:coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
```

#### Task 2: Add Security Scanning

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Check for secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
```

#### Task 3: Build and Push Docker Image

```yaml
# .github/workflows/build.yml
name: Build and Push

on:
  push:
    branches: [ main ]
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ secrets.DOCKER_USERNAME }}/myapp
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/myapp:buildcache
          cache-to: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/myapp:buildcache,mode=max
      
      - name: Sign image
        uses: sigstore/cosign-installer@v2
      
      - name: Sign container image
        run: |
          cosign sign --yes ${{ secrets.DOCKER_USERNAME }}/myapp@${{ steps.meta.outputs.digest }}
        env:
          COSIGN_EXPERIMENTAL: 1
```

#### Task 4: Deploy to Kubernetes

```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'latest'
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=./kubeconfig
      
      - name: Deploy to Kubernetes
        run: |
          export KUBECONFIG=./kubeconfig
          kubectl set image deployment/myapp \
            myapp=${{ secrets.DOCKER_USERNAME }}/myapp:${{ github.sha }} \
            -n production
          kubectl rollout status deployment/myapp -n production
```

### Verification

```bash
# Check GitHub Actions runs
# View workflow runs in GitHub
# Verify tests pass
# Check Docker image in registry
# Verify deployment in Kubernetes
```

### Challenges

1. Add multi-environment deployment (dev, staging, prod)
2. Implement canary deployment
3. Add automated rollback on failure
4. Set up notification on deployment

## 🔄 Lab 2: GitLab CI Multi-Stage Pipeline

### Objective
Build a comprehensive GitLab CI pipeline with multiple stages.

### Prerequisites
- GitLab account
- GitLab repository
- Docker installed (for local testing)

### Tasks

#### Task 1: Basic Multi-Stage Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - security
  - package
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

build:
  stage: build
  image: node:18
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run test
    - npm run test:coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

security:
  stage: security
  image: node:18
  script:
    - npm audit --audit-level=high
    - npm run lint
  allow_failure: false

package:
  stage: package
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker build -t $CI_REGISTRY_IMAGE:latest .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main

deploy:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - kubectl rollout status deployment/myapp
  environment:
    name: production
    url: https://myapp.example.com
  only:
    - main
  when: manual
```

#### Task 2: Parallel Jobs

```yaml
# .gitlab-ci.yml (continued)
test:unit:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run test:unit
  parallel:
    matrix:
      - NODE_VERSION: ["16.x", "18.x", "20.x"]

test:integration:
  stage: test
  image: node:18
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
  script:
    - npm ci
    - npm run test:integration
  only:
    - main
    - merge_requests

test:e2e:
  stage: test
  image: cypress/browsers:latest
  script:
    - npm ci
    - npm run test:e2e
  artifacts:
    when: always
    paths:
      - cypress/screenshots
      - cypress/videos
  only:
    - main
```

#### Task 3: Conditional Jobs

```yaml
# .gitlab-ci.yml (continued)
deploy:staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n staging
  environment:
    name: staging
  only:
    - develop

deploy:production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n production
  environment:
    name: production
  only:
    - main
  when: manual
  allow_failure: false
```

### Verification

```bash
# Check GitLab CI pipeline
# View pipeline in GitLab UI
# Verify all stages pass
# Check artifacts
# Test deployment
```

### Challenges

1. Add performance testing
2. Implement blue-green deployment
3. Add database migrations
4. Set up notification webhooks

## 🔄 Lab 3: Jenkins Declarative Pipeline

### Objective
Create a Jenkins declarative pipeline with multiple stages.

### Prerequisites
- Jenkins installed
- Docker plugin
- Kubernetes plugin
- Git plugin

### Tasks

#### Task 1: Basic Declarative Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE = 'myapp'
        KUBERNETES_NAMESPACE = 'production'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results.xml'
                    publishTestResults testResultsPattern: 'test-results.xml'
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'npm audit --audit-level=high'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def image = docker.build("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                    image.push()
                    image.push("latest")
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl set image deployment/myapp \
                      myapp=${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${env.BUILD_NUMBER} \
                      -n ${KUBERNETES_NAMESPACE}
                    kubectl rollout status deployment/myapp -n ${KUBERNETES_NAMESPACE}
                """
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: "Deployment successful: ${env.BUILD_URL}"
            )
        }
        failure {
            echo 'Pipeline failed!'
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: "Deployment failed: ${env.BUILD_URL}"
            )
        }
        always {
            cleanWs()
        }
    }
}
```

#### Task 2: Parallel Execution

```groovy
// Jenkinsfile (continued)
stage('Parallel Tests') {
    parallel {
        stage('Unit Tests') {
            steps {
                sh 'npm run test:unit'
            }
        }
        stage('Integration Tests') {
            steps {
                sh 'npm run test:integration'
            }
        }
        stage('E2E Tests') {
            steps {
                sh 'npm run test:e2e'
            }
        }
    }
}
```

#### Task 3: Conditional Deployment

```groovy
// Jenkinsfile (continued)
stage('Deploy') {
    when {
        anyOf {
            branch 'main'
            branch 'develop'
        }
    }
    steps {
        script {
            if (env.BRANCH_NAME == 'main') {
                sh 'kubectl apply -f k8s/production/'
            } else {
                sh 'kubectl apply -f k8s/staging/'
            }
        }
    }
}
```

### Verification

```bash
# Check Jenkins pipeline
# View console output
# Verify all stages pass
# Check deployment
```

### Challenges

1. Add approval gates
2. Implement rollback mechanism
3. Add performance testing
4. Set up notification channels

## 🔄 Lab 4: Argo CD GitOps Deployment

### Objective
Set up GitOps workflow with Argo CD.

### Prerequisites
- Kubernetes cluster
- kubectl configured
- Git repository

### Tasks

#### Task 1: Install Argo CD

```bash
# Install Argo CD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods
kubectl wait --for=condition=ready pod --all -n argocd --timeout=300s

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

#### Task 2: Create Application

```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

```bash
# Apply application
kubectl apply -f application.yaml

# View application
kubectl get applications -n argocd
argocd app get myapp
```

#### Task 3: Sync Strategy

```yaml
# application.yaml (with sync strategy)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

#### Task 4: Multi-Environment Setup

```yaml
# application-dev.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
spec:
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: develop
    path: k8s/overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: development

---
# application-prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
spec:
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: false
      selfHeal: false
```

### Verification

```bash
# Check Argo CD UI
# Verify application synced
# Check Kubernetes resources
kubectl get all -n production
```

### Challenges

1. Set up application sets
2. Implement sync waves
3. Add health checks
4. Configure notifications

## ✅ Completion Checklist

- [ ] Created GitHub Actions pipeline
- [ ] Built GitLab CI pipeline
- [ ] Set up Jenkins pipeline
- [ ] Configured Argo CD
- [ ] Tested deployments
- [ ] Documented workflows
- [ ] Completed challenges

---

**Next**: Complete Kubernetes labs and AWS labs for advanced practice.

**Remember**: CI/CD is about automation and reliability. Practice with real projects, test thoroughly, and continuously improve your pipelines. Good CI/CD practices enable fast, reliable software delivery.
