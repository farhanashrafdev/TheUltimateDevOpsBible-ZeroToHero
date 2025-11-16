# Jenkins: Complete Guide to Self-Hosted CI/CD

## 🎯 Introduction

Jenkins is an open-source automation server that enables continuous integration and continuous delivery (CI/CD). It's highly extensible through plugins and supports both declarative and scripted pipelines.

### Why Jenkins?

**Key Benefits**:
- **Self-Hosted**: Full control over your CI/CD infrastructure
- **Extensive Plugin Ecosystem**: 1,800+ plugins available
- **Highly Customizable**: Supports complex workflows
- **Free and Open Source**: No licensing costs
- **Mature and Stable**: Battle-tested in production
- **Multi-Platform**: Runs on Windows, Linux, macOS
- **Distributed Builds**: Scale with agent nodes

### Use Cases

1. **Enterprise CI/CD**: Large-scale automation
2. **Complex Workflows**: Multi-stage, conditional pipelines
3. **Legacy System Integration**: Connect with existing tools
4. **Custom Requirements**: Highly specific automation needs
5. **On-Premises Deployments**: Security and compliance requirements

## 📚 Core Concepts

### Jenkins Architecture

```
Jenkins Controller (Master)
├── Job/Pipeline Definitions
├── Plugin Management
├── User Management
└── Build Queue
    │
    ├── Agent Node 1 (Linux)
    ├── Agent Node 2 (Windows)
    └── Agent Node 3 (Docker)
```

### Pipeline Types

1. **Declarative Pipeline**: Structured, easier to read
2. **Scripted Pipeline**: More flexible, Groovy-based
3. **Freestyle Jobs**: UI-based configuration

## 📝 Complete Pipeline Examples

### Declarative Pipeline (Recommended)

```groovy
pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
        ansiColor('xterm')
        skipStagesAfterUnstable()
    }
    
    environment {
        NODE_VERSION = '20'
        REGISTRY = 'registry.example.com'
        IMAGE_NAME = "${env.JOB_NAME.toLowerCase()}"
        KUBECONFIG = credentials('kubeconfig')
    }
    
    tools {
        nodejs 'NodeJS-20'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm ci'
                sh 'npm run lint'
                sh 'npm run format:check'
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
        
        stage('Build') {
            steps {
                script {
                    def imageTag = "${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                    def imageLatest = "${env.REGISTRY}/${env.IMAGE_NAME}:latest"
                    
                    sh """
                        docker build -t ${imageTag} -t ${imageLatest} .
                        docker push ${imageTag}
                        docker push ${imageLatest}
                    """
                    
                    env.IMAGE_TAG = imageTag
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'trivy image ${env.IMAGE_TAG}'
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    sh """
                        kubectl --kubeconfig=${KUBECONFIG} set image \
                            deployment/app app=${env.IMAGE_TAG} \
                            -n staging
                        kubectl --kubeconfig=${KUBECONFIG} rollout status \
                            deployment/app -n staging
                    """
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                script {
                    sh """
                        kubectl --kubeconfig=${KUBECONFIG} set image \
                            deployment/app app=${env.IMAGE_TAG} \
                            -n production
                        kubectl --kubeconfig=${KUBECONFIG} rollout status \
                            deployment/app -n production
                    """
                }
            }
        }
    }
    
    post {
        success {
            slackSend(
                color: 'good',
                message: "Build ${env.BUILD_NUMBER} succeeded!",
                channel: '#deployments'
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build ${env.BUILD_NUMBER} failed!",
                channel: '#deployments'
            )
        }
        always {
            cleanWs()
            archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
        }
    }
}
```

### Scripted Pipeline (Advanced)

```groovy
node {
    def dockerImage
    
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build') {
        dockerImage = docker.build("myapp:${env.BUILD_NUMBER}")
    }
    
    stage('Test') {
        dockerImage.inside {
            sh 'npm test'
        }
    }
    
    stage('Push') {
        docker.withRegistry('https://registry.example.com', 'registry-credentials') {
            dockerImage.push()
            dockerImage.push('latest')
        }
    }
    
    stage('Deploy') {
        sh 'kubectl set image deployment/app app=myapp:${env.BUILD_NUMBER}'
    }
}
```

### Multi-Branch Pipeline

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh './build.sh'
            }
        }
        
        stage('Test') {
            steps {
                sh './test.sh'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

## 🔧 Advanced Features

### Matrix Strategy

```groovy
pipeline {
    agent any
    
    stages {
        stage('Test Matrix') {
            matrix {
                axes {
                    axis {
                        name 'NODE_VERSION'
                        values '16', '18', '20'
                    }
                    axis {
                        name 'OS'
                        values 'ubuntu', 'alpine'
                    }
                }
                stages {
                    stage('Test') {
                        steps {
                            sh "docker run node:${NODE_VERSION}-${OS} npm test"
                        }
                    }
                }
            }
        }
    }
}
```

### Parallel Execution

```groovy
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

### Credentials Management

```groovy
pipeline {
    agent any
    
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Deploy') {
            steps {
                withCredentials([
                    string(credentialsId: 'api-key', variable: 'API_KEY'),
                    usernamePassword(
                        credentialsId: 'docker-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo $API_KEY'
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                }
            }
        }
    }
}
```

### Shared Libraries

**vars/deploy.groovy**:
```groovy
def call(Map config) {
    sh """
        kubectl set image deployment/${config.deployment} \
            ${config.container}=${config.image} \
            -n ${config.namespace}
        kubectl rollout status deployment/${config.deployment} \
            -n ${config.namespace}
    """
}
```

**Usage**:
```groovy
@Library('shared-lib') _

pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                deploy([
                    deployment: 'app',
                    container: 'app',
                    image: 'myapp:latest',
                    namespace: 'production'
                ])
            }
        }
    }
}
```

### Docker Integration

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    def customImage = docker.build(
                        "myapp:${env.BUILD_NUMBER}",
                        "--build-arg VERSION=${env.BUILD_NUMBER} ."
                    )
                    customImage.push()
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    docker.image('node:20').inside {
                        sh 'npm test'
                    }
                }
            }
        }
    }
}
```

### Kubernetes Integration

```groovy
pipeline {
    agent any
    
    stages {
        stage('Deploy to K8s') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl set image deployment/app \
                        app=myapp:${BUILD_NUMBER} \
                        -n production
                    kubectl rollout status deployment/app -n production
                '''
            }
        }
    }
}
```

## 🎯 Best Practices

### 1. Use Jenkinsfile

```groovy
// Version control your pipelines
// Store in repository root or .jenkins/ directory
```

### 2. Use Declarative Pipelines

```groovy
// Easier to read and maintain
// Better error handling
pipeline {
    // ...
}
```

### 3. Secure Credentials

```groovy
// Never hardcode secrets
// Use Jenkins credential store
withCredentials([string(credentialsId: 'secret', variable: 'SECRET')]) {
    sh 'echo $SECRET'
}
```

### 4. Clean Up Workspace

```groovy
post {
    always {
        cleanWs()
    }
}
```

### 5. Use Agents Efficiently

```groovy
pipeline {
    agent {
        label 'docker'
    }
    // Or
    agent {
        docker {
            image 'node:20'
            reuseNode true
        }
    }
}
```

## 📊 Notifications

### Slack Integration

```groovy
post {
    success {
        slackSend(
            color: 'good',
            message: "Build ${env.BUILD_NUMBER} succeeded!",
            channel: '#deployments'
        )
    }
    failure {
        slackSend(
            color: 'danger',
            message: "Build ${env.BUILD_NUMBER} failed!",
            channel: '#deployments'
        )
    }
}
```

### Email Notifications

```groovy
post {
    failure {
        emailext(
            subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Build failed. Check console output.",
            to: "team@example.com"
        )
    }
}
```

## 🔍 Debugging

### Enable Debug Mode

```groovy
pipeline {
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
    stages {
        stage('Debug') {
            steps {
                sh 'set +x'  // Show commands
                sh 'env | sort'
                sh 'pwd'
            }
        }
    }
}
```

## ✅ Mastery Checklist

- [ ] Create declarative pipeline
- [ ] Use parallel execution
- [ ] Manage credentials securely
- [ ] Integrate with Docker
- [ ] Deploy to Kubernetes
- [ ] Set up notifications
- [ ] Use shared libraries
- [ ] Configure agents
- [ ] Optimize build performance
- [ ] Debug failed builds

---

**Next Steps**:
- Learn [Argo CD](./argo-cd.md) for GitOps
- Explore [GitHub Actions](./github-actions.md) for cloud CI/CD
- Master [GitLab CI](./gitlab-ci.md) for integrated platform

**Remember**: Jenkins is powerful but requires maintenance. Use Jenkinsfile for version control, secure credentials properly, and monitor your Jenkins instance regularly.
