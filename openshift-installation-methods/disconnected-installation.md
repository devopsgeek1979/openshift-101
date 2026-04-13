# ✨ OpenShift Disconnected Installation

Disconnected or air-gapped OpenShift installations are deployed in environments without internet access. This requires mirroring container images and operators to a local registry.

## 🔹 Prerequisites

- Local container registry (e.g., Red Hat Quay, Artifactory)
- Mirror registry host with sufficient storage
- OpenShift installer and CLI tools
- Red Hat pull secret
- Access to Red Hat network for initial setup

## 🔹 Architecture

### 📌 Components

- **Mirror Registry**: Stores OpenShift and application images
- **Mirror Host**: Server with internet access for downloading images
- **Disconnected Cluster**: OpenShift cluster without internet access
- **Bastion Host**: Optional jump host for management

### 📌 Network Requirements

- Mirror registry accessible from cluster nodes
- NTP synchronization
- DNS resolution for internal services
- Firewall rules for registry access

## 🔹 Installation Steps

### 📌 Phase 1: Prepare Mirror Registry

1. Set up mirror registry on a host with internet access:

   ```bash
   # Install podman or docker
   dnf install -y podman

   # Run mirror registry
   podman run -d -p 5000:5000 --name mirror-registry \
     -v /opt/registry:/var/lib/registry \
     registry:2
   ```

2. Configure registry authentication (optional for disconnected)

### 📌 Phase 2: Mirror OpenShift Images

1. Download OpenShift release images:

   ```bash
   export OCP_RELEASE=4.12.0
   export LOCAL_REGISTRY='mirror.example.com:5000'
   export LOCAL_REPOSITORY='ocp4/openshift4'
   export PRODUCT_REPO='openshift-release-dev'
   export LOCAL_SECRET_JSON='/path/to/pull-secret'
   export RELEASE_NAME="ocp-release"

   oc adm release mirror -a ${LOCAL_SECRET_JSON} \
     --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-x86_64 \
     --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} \
     --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-x86_64
   ```

2. Mirror additional images (operators, samples):

   ```bash
   # Mirror Operator Lifecycle Manager (OLM)
   oc adm catalog mirror \
     registry.redhat.io/redhat/redhat-operator-index:v4.12 \
     ${LOCAL_REGISTRY} \
     -a ${LOCAL_SECRET_JSON}

   # Mirror sample images
   oc adm release mirror -a ${LOCAL_SECRET_JSON} \
     --from=quay.io/openshift-release-dev/ocp-v4.0-art-dev \
     --to=${LOCAL_REGISTRY}/ocp4/openshift4 \
     --to-release-image=${LOCAL_REGISTRY}/ocp4/openshift4:4.12.0-x86_64
   ```

### 📌 Phase 3: Create Installation Config

1. Create install-config.yaml with local registry:

   ```yaml
   apiVersion: v1
   baseDomain: example.com
   compute:
   - hyperthreading: Enabled
     name: worker
     replicas: 3
   controlPlane:
     hyperthreading: Enabled
     name: master
     replicas: 3
   metadata:
     name: disconnected-cluster
   imageContentSources:
   - mirrors:
     - mirror.example.com:5000/ocp4/openshift4
     source: quay.io/openshift-release-dev/ocp-release
   - mirrors:
     - mirror.example.com:5000/ocp4/openshift4
     source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
   networking:
     clusterNetwork:
     - cidr: 10.128.0.0/14
       hostPrefix: 23
     networkType: OpenShiftSDN
     serviceNetwork:
     - 172.30.0.0/16
   platform:
     none: {}
   pullSecret: '{"auths": {...}}'
   sshKey: 'ssh-ed25519 AAAA...'
   additionalTrustBundle: |
     -----BEGIN CERTIFICATE-----
     # Mirror registry certificate
     -----END CERTIFICATE-----
   ```

2. Add mirror registry certificate to additionalTrustBundle

### 📌 Phase 4: Install Cluster

1. Generate ignition configs:

   ```bash
   ./openshift-install create ignition-configs --dir=.
   ```

2. Boot nodes with ignition files

3. Monitor installation:

   ```bash
   ./openshift-install wait-for bootstrap-complete --dir=.
   ./openshift-install wait-for install-complete --dir=.
   ```

### 📌 Phase 5: Configure Cluster

1. Disable default OperatorHub sources:

   ```bash
   oc patch OperatorHub cluster --type json \
     -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": true}]'
   ```

2. Create local catalog sources:

   ```bash
   oc apply -f - <<EOF
   apiVersion: operators.coreos.com/v1alpha1
   kind: CatalogSource
   metadata:
     name: my-operator-catalog
     namespace: openshift-marketplace
   spec:
     sourceType: grpc
     image: mirror.example.com:5000/olm/redhat-operator-index:v4.12
     displayName: My Operator Catalog
     publisher: Red Hat
   EOF
   ```

## 🔹 Operator Management

### 📌 Mirror Specific Operators

```bash
# Mirror an operator
oc adm catalog mirror \
  registry.redhat.io/redhat/certified-operator-index:v4.12 \
  mirror.example.com:5000 \
  -a ${LOCAL_SECRET_JSON} \
  --index-filter-by-os='linux/amd64'
```

### 📌 Install Operators

```bash
# Create subscription
oc apply -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: my-operator
  namespace: openshift-operators
spec:
  channel: stable
  name: my-operator
  source: my-operator-catalog
  sourceNamespace: openshift-marketplace
EOF
```

## 🔹 Application Deployment

### 📌 Mirror Application Images

```bash
# Mirror application images
oc image mirror \
  registry.example.com/my-app:latest \
  mirror.example.com:5000/my-app:latest \
  -a ${LOCAL_SECRET_JSON}
```

### 📌 Update Image References

Update deployments to use mirrored images:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: mirror.example.com:5000/my-app:latest
```

## 🔹 Maintenance

### 📌 Update Mirror Registry

1. Pull latest images from Red Hat
2. Mirror to local registry
3. Update image references in cluster

### 📌 Certificate Management

- Renew mirror registry certificates
- Update additionalTrustBundle in cluster
- Restart affected pods

### 📌 Storage Management

- Monitor registry storage usage
- Clean up old images
- Plan for storage expansion

## 🔹 Troubleshooting

### 📌 Image Pull Errors

- Verify mirror registry connectivity
- Check image references
- Validate certificates

### 📌 Operator Installation Failures

- Check catalog source status
- Verify mirrored operator images
- Review operator logs

### 📌 Network Issues

- Test connectivity to mirror registry
- Check DNS resolution
- Validate firewall rules

### 📌 Certificate Issues

- Update additionalTrustBundle
- Restart affected services
- Check certificate validity

## 🔹 Security Considerations

- Secure mirror registry access
- Use TLS certificates
- Implement access controls
- Regular security updates

## 🔹 Best Practices

- Plan storage requirements
- Implement backup strategies
- Document image mappings
- Regular maintenance schedules
- Monitor registry health
