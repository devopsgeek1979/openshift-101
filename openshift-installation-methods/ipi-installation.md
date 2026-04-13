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

## Failure Scenarios and Troubleshooting

### Scenario 1: Bootstrap Node Fails to Start

**Symptoms:**

- Bootstrap node doesn't join the cluster
- Installation hangs at "Waiting for bootstrap completion"

**Possible Causes:**

- Network connectivity issues
- Invalid ignition config
- Resource constraints

**Troubleshooting Steps:**

1. Check bootstrap logs:

   ```bash
   openshift-install gather bootstrap --dir=.
   journalctl -u bootkube.service
   ```

2. Verify network configuration:

   ```bash
   ip addr show
   ip route show
   ```

3. Validate ignition config:

   ```bash
   cat bootstrap.ign | jq .storage.files
   ```

**Resolution:**

- Check cloud provider security groups
- Ensure proper DNS resolution
- Verify bootstrap ignition config

### Scenario 2: API Server Not Accessible

**Symptoms:**

- `oc login` fails
- API server returns connection refused

**Possible Causes:**

- Load balancer misconfiguration
- Certificate issues
- Firewall blocking traffic

**Troubleshooting Steps:**

1. Check load balancer status:

   ```bash
   aws elb describe-load-balancers --load-balancer-name <lb-name>
   ```

2. Verify certificates:

   ```bash
   oc get secrets -n openshift-kube-apiserver
   ```

3. Test connectivity:

   ```bash
   curl -k https://api.cluster.example.com:6443/version
   ```

**Resolution:**

- Recreate load balancer
- Regenerate certificates
- Update security groups

### Scenario 3: Nodes Fail to Join Cluster

**Symptoms:**

- Worker nodes show NotReady status
- Kubelet fails to start

**Possible Causes:**

- Invalid kubeconfig
- Network policy blocking traffic
- Resource exhaustion

**Troubleshooting Steps:**

1. Check node status:

   ```bash
   oc get nodes
   oc describe node <node-name>
   ```

2. Review kubelet logs:

   ```bash
   journalctl -u kubelet -f
   ```

3. Verify network connectivity:

   ```bash
   oc rsh <node-name> curl -k https://api.cluster.example.com:6443
   ```

**Resolution:**

- Regenerate node ignition configs
- Check network policies
- Scale up node resources

### Scenario 4: Installation Times Out

**Symptoms:**

- Installation exceeds time limits
- Bootstrap process doesn't complete

**Possible Causes:**

- Slow network connectivity
- Resource constraints
- DNS issues

**Troubleshooting Steps:**

1. Monitor installation progress:

   ```bash
   openshift-install wait-for bootstrap-complete --log-level debug --dir=.
   ```

2. Check resource usage:

   ```bash
   top
   iostat -x 1
   ```

3. Validate DNS:

   ```bash
   nslookup api.cluster.example.com
   ```

**Resolution:**

- Increase timeout values
- Optimize network settings
- Ensure adequate resources

### Common Troubleshooting Questions

1. **Q: Bootstrap node shows "failed to start" error?**
   A: Check ignition config validity and cloud provider permissions.

2. **Q: API server certificate errors after installation?**
   A: Regenerate certificates using `oc adm certificates regenerate`.

3. **Q: Worker nodes stuck in Pending state?**
   A: Verify machine API and cloud provider quotas.

4. **Q: Installation fails with "no available subnets" error?**
   A: Check VPC/subnet configuration and availability zones.

5. **Q: Load balancer health checks failing?**
   A: Verify security groups allow health check traffic.

## Best Practices for IPI Troubleshooting

- Always collect bootstrap logs before cleanup
- Use `--log-level debug` for detailed output
- Verify cloud provider limits and quotas
- Test network connectivity between all components
- Keep installation artifacts for post-mortem analysis
