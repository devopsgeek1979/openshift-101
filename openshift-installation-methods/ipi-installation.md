# OpenShift Installer-Provisioned Infrastructure (IPI) Installation

Installer-Provisioned Infrastructure (IPI) is the default installation method for OpenShift on supported cloud platforms. The installer creates the necessary infrastructure components automatically.

## Supported Platforms

- Amazon Web Services (AWS)
- Microsoft Azure
- Google Cloud Platform (GCP)
- Red Hat OpenStack Platform
- VMware vSphere
- Bare Metal (with assisted installer)

## Prerequisites

- Valid cloud account with appropriate permissions
- Red Hat OpenShift subscription
- OpenShift CLI (`oc`) and installer downloaded
- SSH key pair

## Installation Steps

### 1. Download OpenShift Installer

```bash
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
tar -xzf openshift-install-linux.tar.gz
```

### 2. Create Install Configuration

Generate a basic install-config.yaml:

```bash
./openshift-install create install-config --dir=.
```

Or create manually:

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
  name: my-cluster
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  aws:  # or azure, gcp, etc.
    region: us-east-1
    baseDomainResourceGroupName: <resource-group>
pullSecret: '{"auths": {...}}'
sshKey: 'ssh-ed25519 AAAA...'
```

### 3. Deploy the Cluster

```bash
./openshift-install create cluster --dir=.
```

The installer will:

- Create cloud resources (VPCs, subnets, instances, load balancers, etc.)
- Generate ignition configs
- Boot the instances
- Configure the cluster

### 4. Monitor Installation

```bash
./openshift-install wait-for bootstrap-complete --dir=.
./openshift-install wait-for install-complete --dir=.
```

### 5. Access the Cluster

```bash
export KUBECONFIG=auth/kubeconfig
oc login -u kubeadmin -p <password>
```

## Platform-Specific Configurations

### AWS

```yaml
platform:
  aws:
    region: us-east-1
    amiID: ami-12345678  # Optional
    serviceEndpoints: []  # For restricted networks
```

### Azure

```yaml
platform:
  azure:
    baseDomainResourceGroupName: openshift-rg
    region: eastus
    cloudName: AzurePublicCloud
```

### GCP

```yaml
platform:
  gcp:
    projectID: my-project
    region: us-central1
```

## Customizations

- Network configuration (CIDR ranges, network type)
- Machine types and sizes
- Additional security groups
- Proxy settings for restricted networks

## Troubleshooting

- Check installer logs in the installation directory
- Verify cloud provider permissions
- Ensure DNS resolution
- Check quota limits in cloud console

## Cleanup

To destroy the cluster:

```bash
./openshift-install destroy cluster --dir=.
```
