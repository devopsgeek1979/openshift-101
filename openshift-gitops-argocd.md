# OpenShift GitOps with ArgoCD

GitOps is a modern approach to continuous deployment that uses Git as the single source of truth for infrastructure and application deployments. ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes and OpenShift.

## Prerequisites

- OpenShift 4.7+ cluster
- OpenShift GitOps Operator installed
- Git repository for application manifests
- ArgoCD CLI (`argocd`) installed (optional)

## Installing OpenShift GitOps Operator

### Method 1: OperatorHub (Recommended)

1. **Install via OperatorHub:**

   ```bash
   # Create namespace
   oc create namespace openshift-gitops

   # Install operator via CLI
   oc apply -f - <<EOF
   apiVersion: operators.coreos.com/v1alpha1
   kind: Subscription
   metadata:
     name: openshift-gitops-operator
     namespace: openshift-operators
   spec:
     channel: stable
     name: openshift-gitops-operator
     source: redhat-operators
     sourceNamespace: openshift-marketplace
   EOF
   ```

2. **Verify installation:**

   ```bash
   oc get pods -n openshift-gitops
   oc get argocd -n openshift-gitops
   ```

### Method 2: Manual Installation

```bash
# Clone ArgoCD operator
git clone https://github.com/argoproj-labs/argocd-operator.git
cd argocd-operator

# Install CRDs and operator
oc apply -f config/crd/bases/
oc apply -f config/rbac/
oc apply -f config/manager/

# Create ArgoCD instance
oc apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: openshift-gitops
spec:
  server:
    route:
      enabled: true
  dex:
    openShiftOAuth: true
EOF
```

## ArgoCD Configuration

### Access ArgoCD Web UI

```bash
# Get ArgoCD route
oc get routes -n openshift-gitops

# Get admin password
oc get secret argocd-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d
```

### Configure SSO with OpenShift

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: openshift-gitops
spec:
  dex:
    openShiftOAuth: true
    image: quay.io/redhat-cop/dex:v2.30.0-openshift
  rbac:
    defaultPolicy: 'role:readonly'
    policy: |
      g, system:cluster-admins, role:admin
      g, argocd-admins, role:admin
  server:
    route:
      enabled: true
      tls:
        termination: reencrypt
        insecureEdgeTerminationPolicy: Redirect
    rbacConfig:
      policy.csv: |
        g, cluster-admins, role:admin
```

## Application Deployment Patterns

### Blue-Green Deployment

Blue-Green deployment is a release strategy that reduces downtime and risk by running two identical production environments called Blue and Green.

#### ArgoCD Blue-Green Setup

1. **Create namespaces:**

   ```bash
   oc create namespace blue
   oc create namespace green
   oc create namespace production  # Points to active environment
   ```

2. **ArgoCD Application for Blue environment:**

   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: myapp-blue
     namespace: openshift-gitops
     labels:
       environment: blue
   spec:
     project: default
     source:
       repoURL: https://github.com/myorg/myapp
       targetRevision: HEAD
       path: blue
     destination:
       server: https://kubernetes.default.svc
       namespace: blue
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
       - CreateNamespace=true
   ```

3. **ArgoCD Application for Green environment:**

   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: myapp-green
     namespace: openshift-gitops
     labels:
       environment: green
   spec:
     project: default
     source:
       repoURL: https://github.com/myorg/myapp
       targetRevision: HEAD
       path: green
     destination:
       server: https://kubernetes.default.svc
       namespace: green
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
       - CreateNamespace=true
   ```

4. **Production Route (Blue-Green Switch):**

   ```yaml
   apiVersion: route.openshift.io/v1
   kind: Route
   metadata:
     name: myapp-production
     namespace: production
   spec:
     to:
       kind: Service
       name: myapp  # This points to either blue or green service
     port:
       targetPort: 8080
   ```

#### Blue-Green Deployment Process

1. **Deploy to Green environment:**

   ```bash
   # Update ArgoCD application to point to new version
   oc patch application myapp-green -n openshift-gitops --type merge -p '{"spec":{"source":{"targetRevision":"v2.0.0"}}}'
   ```

2. **Test Green environment:**

   ```bash
   # Test via direct route to green namespace
   oc get routes -n green
   curl https://myapp-green.example.com/health
   ```

3. **Switch traffic to Green:**

   ```bash
   # Update production service to point to green
   oc patch service myapp -n production --type merge -p '{"spec":{"selector":{"environment":"green"}}}'
   ```

4. **Verify production traffic:**

   ```bash
   # Monitor traffic and application health
   oc get routes -n production
   curl https://myapp-production.example.com/health
   ```

5. **Clean up Blue environment:**

   ```bash
   # Scale down blue deployment
   oc scale deployment myapp -n blue --replicas=0
   ```

### Canary Deployment

Canary deployment gradually rolls out changes to a small subset of users before full deployment.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-canary
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp
    targetRevision: HEAD
    path: canary
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  analysis:
    interval: 5m
    args:
    - name: canary-service
      value: myapp-canary
    - name: stable-service
      value: myapp-stable
    metrics:
    - name: success-rate
      interval: 5m
      successCondition: result[0] >= 0.95
      provider:
        prometheus:
          address: http://prometheus-operated:9090
          query: |
            sum(irate(istio_requests_total{reporter="source",destination_service_name=~"{{args.canary-service}}",response_code!~"5.*"}[5m])) /
            sum(irate(istio_requests_total{reporter="source",destination_service_name=~"{{args.canary-service}}"}[5m]))
```

## Advanced ArgoCD Features

### ApplicationSets for Multi-Environment

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-multienv
  namespace: openshift-gitops
spec:
  generators:
  - list:
      elements:
      - env: dev
        url: https://dev.example.com
      - env: staging
        url: https://staging.example.com
      - env: prod
        url: https://prod.example.com
  template:
    metadata:
      name: 'myapp-{{env}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/myapp
        targetRevision: HEAD
        path: 'environments/{{env}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{env}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### Custom Health Checks

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-custom-health
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp
    targetRevision: HEAD
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  revisionHistoryLimit: 10
```

### Resource Hooks

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-hooks
  namespace: openshift-gitops
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
  source:
    repoURL: https://github.com/myorg/myapp
    targetRevision: HEAD
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  revisionHistoryLimit: 10
```

## Integration with CI/CD

### GitHub Actions Integration

```yaml
name: ArgoCD Sync
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  argocd-sync:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Login to ArgoCD
      uses: argoproj/argo-cd@master
      with:
        server-url: ${{ secrets.ARGOCD_SERVER }}
        auth-token: ${{ secrets.ARGOCD_AUTH_TOKEN }}

    - name: Sync Application
      run: |
        argocd app sync myapp-production
        argocd app wait myapp-production --timeout 600
```

### Jenkins Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy to ArgoCD') {
            steps {
                script {
                    sh '''
                        argocd login ${ARGOCD_SERVER} --username ${ARGOCD_USER} --password ${ARGOCD_PASSWORD}
                        argocd app sync myapp-${BRANCH_NAME}
                        argocd app wait myapp-${BRANCH_NAME} --timeout 600
                    '''
                }
            }
        }
    }
}
```

## Monitoring and Observability

### ArgoCD Metrics

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-metrics-config
  namespace: openshift-gitops
data:
  argocd_metrics: |
    global:
      scrape_interval: 30s
    scrape_configs:
    - job_name: 'argocd'
      static_configs:
      - targets: ['argocd-server-metrics:8082']
```

### Application Health Monitoring

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring-stack
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/monitoring
    targetRevision: HEAD
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Security Best Practices

### RBAC Configuration

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: openshift-gitops
spec:
  rbac:
    defaultPolicy: 'role:readonly'
    policy: |
      g, cluster-admins, role:admin
      g, developers, role:developer
      p, role:developer, applications, get, */*, allow
      p, role:developer, applications, sync, */*, allow
      p, role:developer, applications, update, */*, allow
  server:
    rbacConfig:
      policy.csv: |
        g, cluster-admins, role:admin
        g, developers, role:developer
```

### Secret Management

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: secrets-management
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/secrets
    targetRevision: HEAD
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: secrets
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Troubleshooting ArgoCD

### Common Issues

1. **Application not syncing:**

   ```bash
   # Check application status
   argocd app get myapp

   # Force sync
   argocd app sync myapp

   # Check logs
   oc logs -l app.kubernetes.io/name=argocd-server -n openshift-gitops
   ```

2. **Authentication issues:**

   ```bash
   # Check ArgoCD server logs
   oc logs deployment/argocd-server -n openshift-gitops

   # Verify RBAC configuration
   argocd login <server-url>
   ```

3. **Resource conflicts:**

   ```bash
   # Check for conflicting resources
   oc get events -n <namespace>

   # Use sync options
   argocd app sync myapp --prune --force
   ```

### Performance Tuning

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: openshift-gitops
spec:
  controller:
    processors:
      operation: 10
      status: 20
  server:
    resources:
      limits:
        cpu: 2000m
        memory: 2Gi
      requests:
        cpu: 500m
        memory: 512Mi
  repo:
    resources:
      limits:
        cpu: 1000m
        memory: 1Gi
      requests:
        cpu: 250m
        memory: 256Mi
```

## Integration with OpenShift Pipelines

### Tekton Integration

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: gitops-pipeline
  namespace: openshift-gitops
spec:
  params:
  - name: repo-url
    type: string
  - name: branch
    type: string
  tasks:
  - name: build
    taskRef:
      name: buildah
    params:
    - name: IMAGE
      value: $(params.repo-url):$(params.branch)
  - name: deploy
    taskRef:
      name: argocd-sync
    params:
    - name: application-name
      value: myapp-$(params.branch)
    runAfter:
    - build
```

This comprehensive guide covers GitOps implementation with ArgoCD on OpenShift, including blue-green deployments, canary releases, and integration with CI/CD pipelines.