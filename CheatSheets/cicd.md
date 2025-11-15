# CI/CD Cheat Sheet

## 📚 GitHub Actions

### Basic Workflow
```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: command
```

## 📚 GitLab CI

### Basic Pipeline
```yaml
stages:
  - build
  - test

build:
  stage: build
  script:
    - command
```

## 📚 Jenkins

### Jenkinsfile
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'command' }
        }
    }
}
```

---

**Quick Reference**: Essential CI/CD syntax.

