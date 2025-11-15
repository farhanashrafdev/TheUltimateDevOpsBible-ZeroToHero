# Jenkins

## 🎯 Introduction

Jenkins is an open-source automation server for CI/CD, highly customizable with extensive plugin ecosystem.

## 📚 Core Concepts

### Jenkins Components
- **Jobs**: Build configurations
- **Pipelines**: Code-defined workflows
- **Plugins**: Extend functionality
- **Nodes**: Build agents

## 📝 Jenkinsfile (Declarative)

```groovy
pipeline {
    agent any
    
    stages {
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
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

## 🔧 Key Features

### Parallel Execution
```groovy
parallel {
    stage('Test Unit') {
        steps { sh 'npm run test:unit' }
    }
    stage('Test E2E') {
        steps { sh 'npm run test:e2e' }
    }
}
```

### Environment Variables
```groovy
environment {
    NODE_ENV = 'production'
    API_URL = credentials('api-url')
}
```

## ✅ Best Practices

- Use Jenkinsfile
- Version control pipelines
- Use agents efficiently
- Secure credentials
- Monitor builds
- Clean up workspaces

---

**Next**: Learn Argo CD for GitOps deployments.

