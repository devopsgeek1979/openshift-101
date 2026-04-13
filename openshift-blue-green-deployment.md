# ✨ OpenShift Blue-Green Deployment Strategies

Blue-Green deployment is a release strategy that reduces downtime and risk by running two identical production environments called Blue and Green. At any time, only one of the environments is live, serving all production traffic.

## 🔹 Core Concepts

### 📌 Blue-Green vs Other Strategies

- **Blue-Green**: Two full environments, instant switch
- **Canary**: Gradual rollout to subset of users
- **Rolling**: Update instances incrementally
- **A/B Testing**: Route traffic based on rules

## 🔹 Prerequisites

- OpenShift 4.x cluster
- ArgoCD or OpenShift GitOps (recommended)
- Service Mesh (Istio) for advanced routing
- Monitoring and observability stack

## 🔹 Basic Blue-Green Deployment

### 📌 Manual Implementation

1. **Create namespaces:**

   ```bash
   oc create namespace blue
   oc create namespace green
   oc create namespace production
   ```

2. **Deploy Blue environment:**

   ```yaml
   # blue-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-blue
     namespace: blue
     labels:
       app: myapp
       version: v1.0.0
       environment: blue
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: myapp
         environment: blue
     template:
       metadata:
         labels:
           app: myapp
           environment: blue
       spec:
         containers:
         - name: myapp
           image: myapp:v1.0.0
           ports:
           - containerPort: 8080
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-blue
     namespace: blue
   spec:
     selector:
       app: myapp
       environment: blue
     ports:
     - port: 8080
       targetPort: 8080
   ```

3. **Deploy Green environment:**

   ```yaml
   # green-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-green
     namespace: green
     labels:
       app: myapp
       version: v2.0.0
       environment: green
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: myapp
         environment: green
     template:
       metadata:
         labels:
           app: myapp
           environment: green
       spec:
         containers:
         - name: myapp
           image: myapp:v2.0.0
           ports:
           - containerPort: 8080
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-green
     namespace: green
   spec:
     selector:
       app: myapp
       environment: green
     ports:
     - port: 8080
       targetPort: 8080
   ```

4. **Create production route:**

   ```yaml
   # production-route.yaml
   apiVersion: route.openshift.io/v1
   kind: Route
   metadata:
     name: myapp-production
     namespace: production
   spec:
     to:
       kind: Service
       name: myapp-blue  # Initially points to blue
       namespace: blue
     port:
       targetPort: 8080
     tls:
       termination: edge
   ```

### 📌 Deployment Process

1. **Deploy to Green:**

   ```bash
   oc apply -f green-deployment.yaml
   oc rollout status deployment/myapp-green -n green
   ```

2. **Test Green environment:**

   ```bash
   # Create test route to green
   oc expose service myapp-green -n green --name=myapp-green-test
   curl https://myapp-green-test.example.com/health
   ```

3. **Switch traffic to Green:**

   ```bash
   # Update production route to point to green service
   oc patch route myapp-production -n production --type merge -p '{"spec":{"to":{"name":"myapp-green","namespace":"green"}}}'
   ```

4. **Verify traffic switch:**

   ```bash
   # Check route configuration
   oc get route myapp-production -n production -o yaml

   # Monitor application logs
   oc logs -l app=myapp,environment=green -n green --tail=50
   ```

5. **Clean up Blue environment:**

   ```bash
   # Scale down blue deployment
   oc scale deployment myapp-blue -n blue --replicas=0

   # Optional: Delete blue resources after observation period
   # oc delete namespace blue
   ```

## 🔹 Advanced Blue-Green with Service Mesh

### 📌 Istio Implementation

1. **Install Service Mesh:**

   ```bash
   oc apply -f - <<EOF
   apiVersion: maistra.io/v2
   kind: ServiceMeshControlPlane
   metadata:
     name: basic
     namespace: istio-system
   spec:
     version: v2.3
     tracing:
       sampling: 10000
       type: Jaeger
     addons:
       jaeger:
         name: jaeger
       kiali:
         name: kiali
       grafana:
         name: grafana
   EOF
   ```

2. **Create VirtualService for traffic switching:**

   ```yaml
   apiVersion: networking.istio.io/v1beta1
   kind: VirtualService
   metadata:
     name: myapp-production
     namespace: production
   spec:
     http:
     - route:
       - destination:
           host: myapp-blue.production.svc.cluster.local
         weight: 0
       - destination:
           host: myapp-green.production.svc.cluster.local
         weight: 100
     gateways:
     - istio-system/istio-ingressgateway
   ```

3. **Traffic switching with Istio:**

   ```bash
   # Switch 10% traffic to green
   oc apply -f - <<EOF
   apiVersion: networking.istio.io/v1beta1
   kind: VirtualService
   metadata:
     name: myapp-production
     namespace: production
   spec:
     http:
     - route:
       - destination:
           host: myapp-blue.production.svc.cluster.local
         weight: 90
       - destination:
           host: myapp-green.production.svc.cluster.local
         weight: 10
   EOF

   # Switch all traffic to green
   oc apply -f - <<EOF
   apiVersion: networking.istio.io/v1beta1
   kind: VirtualService
   metadata:
     name: myapp-production
     namespace: production
   spec:
     http:
     - route:
       - destination:
           host: myapp-green.production.svc.cluster.local
         weight: 100
   EOF
   ```

## 🔹 Blue-Green with ArgoCD

### 📌 ArgoCD ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-blue-green
  namespace: openshift-gitops
spec:
  generators:
  - list:
      elements:
      - environment: blue
        version: v1.0.0
      - environment: green
        version: v2.0.0
  template:
    metadata:
      name: 'myapp-{{environment}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/myapp
        targetRevision: '{{version}}'
        path: 'deploy/{{environment}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{environment}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### 📌 Automated Blue-Green Pipeline

```yaml
# blue-green-pipeline.yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: blue-green-pipeline
  namespace: openshift-gitops
spec:
  params:
  - name: new-version
    type: string
  - name: environment
    type: string
  workspaces:
  - name: shared-workspace
  tasks:
  - name: deploy-to-green
    taskRef:
      name: argocd-sync
    params:
    - name: application-name
      value: myapp-green
    - name: revision
      value: $(params.new-version)
  - name: test-green
    taskRef:
      name: test-application
    params:
    - name: environment
      value: green
    runAfter:
    - deploy-to-green
  - name: switch-traffic
    taskRef:
      name: switch-route
    params:
    - name: target-environment
      value: green
    runAfter:
    - test-green
  - name: cleanup-blue
    taskRef:
      name: cleanup-environment
    params:
    - name: environment
      value: blue
    runAfter:
    - switch-traffic
```

## 🔹 Database Considerations

### 📌 Database Migration Strategy

1. **Backward Compatible Changes:**

   ```sql
   -- Safe changes that don't break existing schema
   ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT FALSE;
   CREATE INDEX idx_users_email ON users(email);
   ```

2. **Schema Migration with Blue-Green:**

   ```bash
   # Deploy green environment with new schema
   oc apply -f green-deployment.yaml
   oc apply -f green-database-migration.yaml

   # Run database migration
   oc run migration-job --image=migration-tool --restart=Never --rm -i --tty

   # Test green environment
   # Switch traffic
   # Clean up blue
   ```

3. **Database Connection Handling:**

   ```yaml
   # Use different database schemas for blue/green
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: db-config-blue
     namespace: blue
   data:
     DB_SCHEMA: myapp_blue
   ---
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: db-config-green
     namespace: green
   data:
     DB_SCHEMA: myapp_green
   ```

## 🔹 Monitoring and Observability

### 📌 Blue-Green Metrics

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: blue-green-alerts
  namespace: monitoring
spec:
  groups:
  - name: blue-green
    rules:
    - alert: BlueGreenTrafficSwitch
      expr: |
        rate(http_requests_total{environment="green"}[5m]) /
        rate(http_requests_total{environment="blue"}[5m]) > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Traffic switching to green environment detected"
    - alert: BlueGreenDeploymentFailure
      expr: |
        kube_deployment_status_replicas_unavailable{deployment=~"myapp-(blue|green)"} > 0
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: "Blue-Green deployment has unavailable replicas"
```

### 📌 Application Health Checks

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp-health-blue
  namespace: blue
spec:
  to:
    kind: Service
    name: myapp-blue
  path: /health
  port:
    targetPort: 8080
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp-health-green
  namespace: green
spec:
  to:
    kind: Service
    name: myapp-green
  path: /health
  port:
    targetPort: 8080
```

## 🔹 Rollback Strategy

### 📌 Automated Rollback

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: rollback-pipeline
  namespace: openshift-gitops
spec:
  tasks:
  - name: check-health
    taskRef:
      name: health-check
    params:
    - name: environment
      value: green
  - name: rollback-traffic
    taskRef:
      name: switch-route
    params:
    - name: target-environment
      value: blue
    runAfter:
    - check-health
    conditions:
    - conditionRef: health-check-failed
      params:
      - name: status
        value: failed
  - name: scale-down-green
    taskRef:
      name: scale-deployment
    params:
    - name: environment
      value: green
      replicas: 0
    runAfter:
    - rollback-traffic
```

### 📌 Manual Rollback

```bash
# Quick rollback to blue
oc patch route myapp-production -n production --type merge -p '{"spec":{"to":{"name":"myapp-blue","namespace":"blue"}}}'

# Scale up blue if needed
oc scale deployment myapp-blue -n blue --replicas=3

# Scale down green
oc scale deployment myapp-green -n green --replicas=0
```

## 🔹 Security Considerations

### 📌 Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: blue-green-isolation
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: blue
    - namespaceSelector:
        matchLabels:
          environment: green
    ports:
    - protocol: TCP
      port: 8080
```

### 📌 RBAC for Blue-Green

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: blue-green-deployer
  namespace: production
rules:
- apiGroups: ["route.openshift.io"]
  resources: ["routes"]
  verbs: ["get", "patch", "update"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "scale"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: blue-green-deployer-binding
  namespace: production
subjects:
- kind: Group
  name: deployers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: blue-green-deployer
  apiGroup: rbac.authorization.k8s.io
```

## 🔹 Cost Optimization

### 📌 Resource Management

```yaml
# Scale down inactive environment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  namespace: blue
spec:
  replicas: 0  # Scaled down when not active
  template:
    spec:
      containers:
      - name: myapp
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

### 📌 Auto-scaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-green-hpa
  namespace: green
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-green
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## 🔹 Integration with CI/CD

### 📌 GitHub Actions Blue-Green

```yaml
name: Blue-Green Deployment
on:
  push:
    branches: [ main ]

jobs:
  blue-green-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Deploy to Green
      run: |
        oc apply -f green-deployment.yaml
        oc rollout status deployment/myapp-green -n green

    - name: Health Check
      run: |
        curl -f https://myapp-green-test.example.com/health

    - name: Switch Traffic
      run: |
        oc patch route myapp-production -n production --type merge -p '{"spec":{"to":{"name":"myapp-green","namespace":"green"}}}'

    - name: Cleanup Blue
      run: |
        oc scale deployment myapp-blue -n blue --replicas=0
```

This comprehensive guide covers blue-green deployment strategies on OpenShift, from basic implementations to advanced scenarios with service mesh and automated pipelines.
