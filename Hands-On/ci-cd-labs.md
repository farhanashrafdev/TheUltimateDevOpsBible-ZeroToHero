# CI/CD Labs

## 🎯 CI/CD Practical Exercises

### Lab 1: GitHub Actions
**Objective**: Create CI pipeline

**Steps**:
1. Create `.github/workflows/ci.yml`
2. Configure build job
3. Add test job
4. Set up artifacts

**Example**:
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test
```

### Lab 2: GitLab CI
**Objective**: Multi-stage pipeline

**Steps**:
1. Create `.gitlab-ci.yml`
2. Define stages
3. Configure jobs
4. Add deployment

**Example**:
```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - docker build -t app .

test:
  stage: test
  script:
    - docker run app npm test
```

### Lab 3: Jenkins Pipeline
**Objective**: Declarative pipeline

**Steps**:
1. Create Jenkinsfile
2. Configure stages
3. Add post-actions
4. Test pipeline

**Example**:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }
}
```

### Lab 4: Argo CD
**Objective**: GitOps deployment

**Steps**:
1. Install Argo CD
2. Create application
3. Configure sync
4. Deploy application

**Application Example**:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    repoURL: https://github.com/user/repo.git
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

## ✅ Completion Checklist

- [ ] Created CI pipelines
- [ ] Configured CD
- [ ] Tested deployments
- [ ] Documented workflows

---

**Next**: Complete Kubernetes labs.

