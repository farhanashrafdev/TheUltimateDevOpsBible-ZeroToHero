# CI/CD Cheat Sheet - Complete Reference

## 🔄 GitHub Actions

### Workflow Structure

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 0 * * *'  # Daily
  workflow_dispatch:     # Manual trigger

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello"
```

### Common Actions

```yaml
# Checkout
- uses: actions/checkout@v4
- uses: actions/checkout@v4
  with:
    ref: branch-name
    path: custom-path

# Setup Node.js
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'

# Setup Python
- uses: actions/setup-python@v5
  with:
    python-version: '3.11'
    cache: 'pip'

# Setup Docker
- uses: docker/setup-buildx-action@v3
- uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

# Cache
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

### Matrix Strategy

```yaml
strategy:
  matrix:
    node-version: [18, 20]
    os: [ubuntu-latest, windows-latest]
  fail-fast: false
```

### Secrets and Variables

```yaml
env:
  SECRET: ${{ secrets.MY_SECRET }}
  VAR: ${{ vars.MY_VAR }}

steps:
  - run: echo "${{ secrets.API_KEY }}"
```

### Conditions

```yaml
if: github.ref == 'refs/heads/main'
if: success()
if: failure()
if: always()
if: github.event_name == 'pull_request'
```

## 🔧 GitLab CI

### Pipeline Structure

```yaml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG

before_script:
  - echo "Before script"

build:
  stage: build
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - main
    - develop
```

### GitLab CI Variables

```yaml
# Predefined variables
$CI_COMMIT_SHA
$CI_COMMIT_REF_NAME
$CI_PROJECT_NAME
$CI_JOB_NAME
$CI_PIPELINE_ID
$CI_REGISTRY
$CI_REGISTRY_IMAGE
```

### Rules

```yaml
rules:
  - if: $CI_COMMIT_BRANCH == "main"
  - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  - if: $CI_COMMIT_TAG
    when: manual
```

### Cache

```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .cache/
```

### Artifacts

```yaml
artifacts:
  paths:
    - dist/
  expire_in: 1 week
  when: on_success
```

## 🏭 Jenkins

### Declarative Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        NODE_VERSION = '20'
        REGISTRY = 'registry.example.com'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
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
                }
            }
        }
    }
    
    post {
        success {
            slackSend(message: "Build succeeded!")
        }
        failure {
            slackSend(message: "Build failed!")
        }
    }
}
```

### Scripted Pipeline

```groovy
node {
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build') {
        sh 'npm install'
        sh 'npm run build'
    }
}
```

### Jenkinsfile Syntax

```groovy
// Environment
env.NODE_VERSION = '20'

// Credentials
withCredentials([string(credentialsId: 'api-key', variable: 'API_KEY')]) {
    sh "echo $API_KEY"
}

// Parallel
parallel(
    'test': {
        sh 'npm test'
    },
    'lint': {
        sh 'npm run lint'
    }
)
```

## 🔄 CircleCI

### Config Structure

```yaml
version: 2.1

jobs:
  build:
    docker:
      - image: node:20
    steps:
      - checkout
      - run: npm install
      - run: npm test

workflows:
  version: 2
  build-and-test:
    jobs:
      - build
```

## 📦 Common Patterns

### Docker Build and Push

```yaml
# GitHub Actions
- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### Kubernetes Deploy

```yaml
# GitHub Actions
- name: Deploy to K8s
  run: |
    kubectl set image deployment/app app=${{ env.IMAGE_TAG }}
    kubectl rollout status deployment/app
  env:
    KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

### Notifications

```yaml
# Slack
- name: Notify Slack
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Deployment completed'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

## 🚨 Troubleshooting

### Debug Mode

```yaml
# GitHub Actions
- name: Debug
  run: |
    echo "::debug::Debug message"
    echo "::warning::Warning message"
    echo "::error::Error message"
```

### Common Issues

```bash
# Check workflow syntax
# GitHub Actions: Validate YAML
# GitLab CI: CI Lint
# Jenkins: Declarative Linter

# View logs
# Check runner logs
# Check container logs
# Enable debug mode
```

---

**Remember**: Keep pipelines fast, use caching, parallelize when possible, and always test locally first.
