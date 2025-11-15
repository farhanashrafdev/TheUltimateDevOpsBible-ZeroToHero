# Argo Workflows: Complete Workflow Orchestration Guide

## 🎯 Introduction

Argo Workflows is an open-source container-native workflow engine for orchestrating parallel jobs on Kubernetes. It enables you to define complex workflows as Kubernetes custom resources, making it perfect for CI/CD pipelines, data processing, machine learning workflows, and batch jobs.

### Why Argo Workflows?

**Key Benefits**:
- **Kubernetes Native**: Runs on Kubernetes, uses Kubernetes resources
- **DAG Support**: Define complex dependencies with Directed Acyclic Graphs
- **Parallel Execution**: Run multiple steps in parallel
- **Artifacts**: Share data between workflow steps
- **Retries**: Automatic retry with backoff
- **Suspension/Resume**: Pause and resume workflows
- **Web UI**: Visual workflow management
- **Parameterization**: Dynamic workflow execution

### Use Cases

1. **CI/CD Pipelines**: Complex build and deployment workflows
2. **Data Processing**: ETL pipelines, batch jobs
3. **Machine Learning**: Training and inference pipelines
4. **Scheduled Jobs**: Cron-like workflows
5. **Multi-Step Processes**: Complex business logic

## 📚 Core Concepts

### Workflow Components

```
Workflow
├── Entrypoint (starting template)
├── Templates (reusable steps)
│   ├── Container templates
│   ├── Script templates
│   ├── Resource templates
│   ├── DAG templates
│   └── Steps templates
├── Arguments (input parameters)
└── Artifacts (data sharing)
```

### Workflow Lifecycle

```
Pending → Running → Succeeded/Failed
              ↓
         Suspended (optional)
```

## 📝 Complete Workflow Examples

### Basic Container Workflow

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-
spec:
  entrypoint: whalesay
  templates:
    - name: whalesay
      container:
        image: docker/whalesay:latest
        command: [cowsay]
        args: ["hello world"]
```

### Multi-Step Workflow

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: steps-example-
spec:
  entrypoint: hello-hello-hello
  templates:
    - name: hello-hello-hello
      steps:
        - - name: hello1
            template: whalesay
            arguments:
              parameters:
                - name: message
                  value: "hello1"
        - - name: hello2
            template: whalesay
            arguments:
              parameters:
                - name: message
                  value: "hello2"
        - - name: hello3
            template: whalesay
            arguments:
              parameters:
                - name: message
                  value: "hello3"
    
    - name: whalesay
      inputs:
        parameters:
          - name: message
      container:
        image: docker/whalesay:latest
        command: [cowsay]
        args: ["{{inputs.parameters.message}}"]
```

### DAG Workflow

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: dag-diamond-
spec:
  entrypoint: diamond
  templates:
    - name: echo
      inputs:
        parameters:
          - name: message
      container:
        image: alpine:3.7
        command: [echo, "{{inputs.parameters.message}}"]
    
    - name: diamond
      dag:
        tasks:
          - name: A
            template: echo
            arguments:
              parameters:
                - name: message
                  value: A
          - name: B
            dependencies: [A]
            template: echo
            arguments:
              parameters:
                - name: message
                  value: B
          - name: C
            dependencies: [A]
            template: echo
            arguments:
              parameters:
                - name: message
                  value: C
          - name: D
            dependencies: [B, C]
            template: echo
            arguments:
              parameters:
                - name: message
                  value: D
```

### Workflow with Artifacts

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: artifact-example-
spec:
  entrypoint: artifact-example
  templates:
    - name: artifact-example
      steps:
        - - name: generate-artifact
            template: whalesay
        - - name: consume-artifact
            template: print-message
            arguments:
              artifacts:
                - name: hello-art
                  from: "{{steps.generate-artifact.outputs.artifacts.hello-art}}"
    
    - name: whalesay
      container:
        image: docker/whalesay:latest
        command: [sh, -c]
        args: ["cowsay hello world | tee /tmp/hello_world.txt"]
      outputs:
        artifacts:
          - name: hello-art
            path: /tmp/hello_world.txt
    
    - name: print-message
      inputs:
        artifacts:
          - name: hello-art
            path: /tmp/message
      container:
        image: alpine:latest
        command: [sh, -c]
        args: ["cat /tmp/message"]
```

### Workflow with Parameters

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: parameter-example-
spec:
  entrypoint: whalesay
  arguments:
    parameters:
      - name: message
        value: hello world
  templates:
    - name: whalesay
      inputs:
        parameters:
          - name: message
      container:
        image: docker/whalesay:latest
        command: [cowsay]
        args: ["{{inputs.parameters.message}}"]
```

### CI/CD Pipeline Workflow

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: cicd-pipeline-
spec:
  entrypoint: pipeline
  arguments:
    parameters:
      - name: git-repo
        value: https://github.com/user/repo.git
      - name: image-tag
        value: latest
  templates:
    - name: pipeline
      dag:
        tasks:
          - name: checkout
            template: git-checkout
            arguments:
              parameters:
                - name: repo
                  value: "{{workflow.parameters.git-repo}}"
          
          - name: build
            dependencies: [checkout]
            template: docker-build
            arguments:
              parameters:
                - name: tag
                  value: "{{workflow.parameters.image-tag}}"
          
          - name: test
            dependencies: [build]
            template: run-tests
          
          - name: security-scan
            dependencies: [build]
            template: trivy-scan
            arguments:
              parameters:
                - name: image
                  value: "myapp:{{workflow.parameters.image-tag}}"
          
          - name: deploy
            dependencies: [test, security-scan]
            template: deploy-k8s
            arguments:
              parameters:
                - name: image
                  value: "myapp:{{workflow.parameters.image-tag}}"
    
    - name: git-checkout
      inputs:
        parameters:
          - name: repo
      container:
        image: alpine/git:latest
        command: [sh, -c]
        args:
          - |
            git clone {{inputs.parameters.repo}} /workspace
            cd /workspace
            ls -la
    
    - name: docker-build
      inputs:
        parameters:
          - name: tag
      container:
        image: docker:latest
        command: [sh, -c]
        args:
          - |
            docker build -t myapp:{{inputs.parameters.tag}} .
            docker push myapp:{{inputs.parameters.tag}}
        volumeMounts:
          - name: docker-sock
            mountPath: /var/run/docker.sock
      volumes:
        - name: docker-sock
          hostPath:
            path: /var/run/docker.sock
    
    - name: run-tests
      container:
        image: node:20
        command: [sh, -c]
        args:
          - |
            npm install
            npm test
    
    - name: trivy-scan
      inputs:
        parameters:
          - name: image
      container:
        image: aquasec/trivy:latest
        command: [trivy]
        args: ["image", "--exit-code", "0", "{{inputs.parameters.image}}"]
    
    - name: deploy-k8s
      inputs:
        parameters:
          - name: image
      container:
        image: bitnami/kubectl:latest
        command: [sh, -c]
        args:
          - |
            kubectl set image deployment/app app={{inputs.parameters.image}}
            kubectl rollout status deployment/app
```

### Workflow with Retries

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: retry-example-
spec:
  entrypoint: retry-container
  templates:
    - name: retry-container
      retryStrategy:
        limit: 5
        retryPolicy: "Always"
        backoff:
          duration: "5s"
          factor: 2
          maxDuration: "3m"
      container:
        image: python:alpine
        command: [sh, -c]
        args: ["python -c 'import random; exit(1 if random.random() > 0.3 else 0)'"]
```

### Workflow with Timeouts

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: timeout-example-
spec:
  entrypoint: sleep
  templates:
    - name: sleep
      activeDeadlineSeconds: 10
      container:
        image: alpine:latest
        command: [sh, -c]
        args: ["sleep 20"]  # Will timeout after 10 seconds
```

### Workflow with Resource Management

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: resource-example-
spec:
  entrypoint: resource-workflow
  templates:
    - name: resource-workflow
      container:
        image: nginx:latest
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

## 🔧 Advanced Features

### Workflow Templates

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: ci-pipeline
spec:
  entrypoint: pipeline
  templates:
    - name: pipeline
      steps:
        - - name: build
            template: build
        - - name: test
            template: test
            dependencies: [build]
```

### Cron Workflows

```yaml
apiVersion: argoproj.io/v1alpha1
kind: CronWorkflow
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  timezone: "America/New_York"
  workflowSpec:
    entrypoint: backup
    templates:
      - name: backup
        container:
          image: backup-tool:latest
          command: [backup.sh]
```

### Workflow with Conditions

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: conditional-
spec:
  entrypoint: conditional-example
  arguments:
    parameters:
      - name: should-run
        value: "true"
  templates:
    - name: conditional-example
      steps:
        - - name: check
            template: check-condition
        - - name: run-if-true
            template: run-task
            when: "{{steps.check.outputs.result}} == true"
    
    - name: check-condition
      container:
        image: alpine:latest
        command: [sh, -c]
        args: ['echo "{{workflow.parameters.should-run}}"']
      outputs:
        parameters:
          - name: result
            valueFrom:
              default: "false"
              path: /tmp/result
    
    - name: run-task
      container:
        image: alpine:latest
        command: [echo, "Task executed"]
```

## 🛠️ CLI Commands

### Workflow Management

```bash
# Submit workflow
argo submit workflow.yaml
argo submit workflow.yaml -p message="hello"

# List workflows
argo list
argo list --status Running
argo list --all-namespaces

# Get workflow
argo get workflow-name
argo get workflow-name -o yaml

# Watch workflow
argo watch workflow-name

# Delete workflow
argo delete workflow-name
argo delete --all

# Retry workflow
argo retry workflow-name

# Suspend/Resume
argo suspend workflow-name
argo resume workflow-name
```

### Workflow Templates

```bash
# Create workflow template
argo template create template.yaml

# List templates
argo template list

# Get template
argo template get template-name
```

### Cron Workflows

```bash
# Create cron workflow
argo cron create cron-workflow.yaml

# List cron workflows
argo cron list

# Delete cron workflow
argo cron delete cron-workflow-name
```

## 🎯 Best Practices

### 1. Use Templates for Reusability

```yaml
templates:
  - name: common-task
    container:
      image: common-image:latest
      # Reusable template
```

### 2. Implement Error Handling

```yaml
retryStrategy:
  limit: 3
  retryPolicy: "OnError"
  backoff:
    duration: "10s"
    factor: 2
```

### 3. Set Resource Limits

```yaml
container:
  resources:
    requests:
      memory: "128Mi"
      cpu: "100m"
    limits:
      memory: "256Mi"
      cpu: "500m"
```

### 4. Use Artifacts for Data Sharing

```yaml
outputs:
  artifacts:
    - name: output-data
      path: /tmp/output
```

### 5. Monitor Workflows

```bash
# Use Argo UI or CLI
argo watch workflow-name
```

## ✅ Mastery Checklist

- [ ] Create basic workflows
- [ ] Use DAG for complex dependencies
- [ ] Implement parallel execution
- [ ] Share artifacts between steps
- [ ] Use parameters for dynamic workflows
- [ ] Implement retries and timeouts
- [ ] Create workflow templates
- [ ] Set up cron workflows
- [ ] Manage resources properly
- [ ] Monitor and debug workflows

---

**Next Steps**:
- Learn [Argo CD](./argo-cd.md) for GitOps
- Explore [Kubernetes Operations](./kubernetes-operations.md)
- Master [CI/CD](./ci-cd.md) integration

**Remember**: Argo Workflows is powerful for complex orchestration. Start with simple workflows, use DAGs for dependencies, and always set resource limits. Monitor workflows and implement proper error handling.
