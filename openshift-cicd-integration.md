# OpenShift CI/CD Integration Guide

This guide covers integrating various CI/CD tools with OpenShift for automated deployments, GitOps workflows, and DevSecOps practices.

## OpenShift Pipelines (Tekton)

### Installation

```bash
# Install OpenShift Pipelines Operator
oc apply -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-pipelines-operator
  namespace: openshift-operators
spec:
  channel: stable
  name: openshift-pipelines-operator-rh
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

### Basic Pipeline for Application Deployment

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: deploy-pipeline
  namespace: openshift-gitops
spec:
  params:
  - name: repo-url
    type: string
  - name: branch
    type: string
  - name: image-tag
    type: string
  workspaces:
  - name: shared-workspace
  tasks:
  - name: clone
    taskRef:
      name: git-clone
    params:
    - name: url
      value: $(params.repo-url)
    - name: revision
      value: $(params.branch)
    workspaces:
    - name: output
      workspace: shared-workspace
  - name: build
    taskRef:
      name: buildah
    params:
    - name: IMAGE
      value: image-registry.openshift-image-registry.svc:5000/$(context.pipelineRun.namespace)/$(context.pipelineRun.name):$(params.image-tag)
    workspaces:
    - name: source
      workspace: shared-workspace
  - name: deploy
    taskRef:
      name: openshift-client
    params:
    - name: SCRIPT
      value: |
        oc apply -f k8s/
        oc rollout status deployment/myapp
    workspaces:
    - name: manifest-dir
      workspace: shared-workspace
```

## Jenkins Integration

### Jenkins on OpenShift

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: ci-cd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      serviceAccountName: jenkins
      containers:
      - name: jenkins
        image: quay.io/openshift/origin-jenkins:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: jenkins-data
          mountPath: /var/lib/jenkins
      volumes:
      - name: jenkins-data
        persistentVolumeClaim:
          claimName: jenkins-pvc
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: ci-cd
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins-role
  namespace: ci-cd
rules:
- apiGroups: [""]
  resources: ["pods", "services", "endpoints"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-role-binding
  namespace: ci-cd
subjects:
- kind: ServiceAccount
  name: jenkins
roleRef:
  kind: Role
  name: jenkins-role
  apiGroup: rbac.authorization.k8s.io
```

### Jenkins Pipeline for Blue-Green Deployment

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: jnlp
    image: quay.io/openshift/origin-jenkins-agent-base:latest
  - name: oc
    image: quay.io/openshift/origin-cli:latest
    command: ['cat']
    tty: true
'''
        }
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                container('oc') {
                    sh 'oc start-build myapp --from-dir=. --wait'
                }
            }
        }
        stage('Deploy to Green') {
            steps {
                container('oc') {
                    sh '''
                        oc apply -f k8s/green/
                        oc rollout status deployment/myapp-green -n green
                    '''
                }
            }
        }
        stage('Test Green') {
            steps {
                container('oc') {
                    sh '''
                        oc expose service myapp-green -n green --name=test-green
                        curl -f https://test-green-green.apps.example.com/health
                    '''
                }
            }
        }
        stage('Switch Traffic') {
            steps {
                container('oc') {
                    sh '''
                        oc patch route myapp-production -n production --type merge -p '{"spec":{"to":{"name":"myapp-green","namespace":"green"}}}'
                    '''
                }
            }
        }
        stage('Cleanup') {
            steps {
                container('oc') {
                    sh '''
                        oc scale deployment myapp-blue -n blue --replicas=0
                    '''
                }
            }
        }
    }
    post {
        failure {
            container('oc') {
                sh '''
                    # Rollback to blue on failure
                    oc patch route myapp-production -n production --type merge -p '{"spec":{"to":{"name":"myapp-blue","namespace":"blue"}}}'
                    oc scale deployment myapp-blue -n blue --replicas=3
                '''
            }
        }
    }
}
```

## GitHub Actions with OpenShift

### GitHub Actions Workflow

```yaml
name: OpenShift CI/CD
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Login to OpenShift
      uses: redhat-actions/oc-login@v1
      with:
        openshift_server_url: ${{ secrets.OPENSHIFT_SERVER }}
        openshift_token: ${{ secrets.OPENSHIFT_TOKEN }}
        namespace: ${{ secrets.OPENSHIFT_NAMESPACE }}

    - name: Build Application
      run: |
        oc start-build myapp --from-dir=. --wait

    - name: Deploy to Development
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: |
        oc apply -f k8s/dev/
        oc rollout status deployment/myapp-dev

    - name: Run Tests
      run: |
        oc run test-runner --image=myapp:latest --restart=Never --rm -i --tty -- oc test
```

## ArgoCD Integration with CI/CD

### Automated Sync Triggers

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: openshift-gitops
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp
    targetRevision: HEAD
    path: deploy/production
    kustomize:
      images:
      - myapp=quay.io/myorg/myapp:${IMAGE_TAG}
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
```

### Webhook Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-webhook-config
  namespace: openshift-gitops
data:
  webhook.github: |
    secret: ${{ secrets.WEBHOOK_SECRET }}
    url: https://argocd-server-openshift-gitops.apps.example.com/api/webhook
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: argocd-webhook
  namespace: openshift-gitops
spec:
  to:
    kind: Service
    name: argocd-server
  port:
    targetPort: https
  tls:
    termination: reencrypt
```

## DevSecOps Integration

### Security Scanning

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: secure-pipeline
  namespace: openshift-gitops
spec:
  tasks:
  - name: clone
    taskRef:
      name: git-clone
  - name: security-scan
    taskRef:
      name: clamav-scan
    runAfter:
    - clone
  - name: sonar-scan
    taskRef:
      name: sonar-scanner
    runAfter:
    - security-scan
  - name: build
    taskRef:
      name: buildah
    runAfter:
    - sonar-scan
  - name: image-scan
    taskRef:
      name: trivy-scan
    runAfter:
    - build
  - name: deploy
    taskRef:
      name: argocd-sync
    runAfter:
    - image-scan
    conditions:
    - conditionRef: security-gate
      params:
      - name: severity
        value: high
```

### Compliance Checks

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: compliance-check
  namespace: openshift-gitops
spec:
  params:
  - name: image-ref
    type: string
  steps:
  - name: check-compliance
    image: quay.io/compliance-operator/compliance-check:latest
    script: |
      #!/bin/bash
      echo "Running compliance checks on $(params.image-ref)"
      # Check for required labels
      # Validate security context
      # Check for vulnerabilities
      # Verify license compliance
```

## Monitoring CI/CD Pipelines

### Prometheus Metrics

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: tekton-monitor
  namespace: openshift-gitops
spec:
  selector:
    matchLabels:
      app: tekton-pipelines-controller
  endpoints:
  - port: metrics
    interval: 30s
```

### Dashboard Integration

```yaml
apiVersion: integreatly.org/v1alpha1
kind: GrafanaDashboard
metadata:
  name: cicd-dashboard
  namespace: openshift-gitops
spec:
  json: |
    {
      "dashboard": {
        "title": "CI/CD Pipeline Dashboard",
        "panels": [
          {
            "title": "Pipeline Success Rate",
            "type": "stat",
            "targets": [
              {
                "expr": "rate(tekton_pipelines_controller_running_pipelines_total[5m])",
                "legendFormat": "Running Pipelines"
              }
            ]
          }
        ]
      }
    }
```

## Multi-Cluster CI/CD

### Cluster Synchronization

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-cluster-cicd
  namespace: openshift-gitops
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          environment: production
  template:
    metadata:
      name: 'cicd-{{name}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/cicd-config
        targetRevision: HEAD
        path: clusters/{{name}}
      destination:
        server: '{{server}}'
        namespace: cicd
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

This comprehensive CI/CD integration guide covers the essential tools and practices for implementing automated deployments on OpenShift, integrating with GitOps workflows and DevSecOps practices.
