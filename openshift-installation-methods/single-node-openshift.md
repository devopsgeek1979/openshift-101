# Single-Node OpenShift (SNO) Installation

Single-Node OpenShift (SNO) is a compact, all-in-one deployment of OpenShift that runs the control plane and worker components on a single node. It's ideal for edge computing, development environments, and resource-constrained deployments.

## Key Features

- All components on one node
- Reduced resource requirements
- Simplified management
- Automatic recovery capabilities
- Support for disconnected environments

## Prerequisites

- Single RHEL 8.6+ or RHCOS server
- Minimum: 8 CPU cores, 32 GB RAM, 200 GB storage
- Recommended: 16 CPU cores, 64 GB RAM, 500 GB storage
- Network connectivity
- Red Hat subscription

## Installation Methods

### Method 1: Assisted Installer

1. Access Red Hat Hybrid Cloud Console
2. Select "Single-node" cluster type
3. Follow the guided installation

### Method 2: Manual Installation

1. Download OpenShift installer:

   ```bash
   wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
   tar -xzf openshift-install-linux.tar.gz
   ```

2. Create install-config.yaml:

   ```yaml
   apiVersion: v1
   baseDomain: example.com
   compute:
   - hyperthreading: Enabled
     name: worker
     replicas: 0  # No separate workers
   controlPlane:
     hyperthreading: Enabled
     name: master
     replicas: 1  # Single control plane
   metadata:
     name: sno-cluster
   networking:
     clusterNetwork:
     - cidr: 10.128.0.0/14
       hostPrefix: 23
     networkType: OpenShiftSDN
     serviceNetwork:
     - 172.30.0.0/16
   platform:
     none: {}  # For bare metal
   pullSecret: '{"auths": {...}}'
   sshKey: 'ssh-ed25519 AAAA...'
   ```

3. Generate ignition config:

   ```bash
   ./openshift-install create single-node-ignition-config --dir=.
   ```

4. Boot the node with the ignition file

5. Monitor installation:

   ```bash
   ./openshift-install wait-for install-complete --dir=.
   ```

## Post-Installation Configuration

### Access the Cluster

```bash
export KUBECONFIG=auth/kubeconfig
oc login -u kubeadmin -p <password>
```

### Enable Workloads on Master

By default, workloads don't run on the master node. To enable:

```bash
oc patch schedulers.config.openshift.io/cluster --type merge --patch '{"spec":{"mastersSchedulable": true}}'
```

### Configure Storage

Set up local storage or external storage solutions.

### Network Configuration

Configure ingress and routes as needed.

## Management and Operations

### Monitoring

SNO includes built-in monitoring. Access via:

```bash
oc get routes -n openshift-monitoring
```

### Updates

SNO supports in-place updates:

```bash
oc adm upgrade
```

### Backup and Recovery

- ETCD is automatically backed up
- Use OADP for application backups
- Node recovery is automatic for most failures

## Scaling

While SNO runs on a single node, you can add worker nodes later:

1. Scale up the worker machineset:

   ```bash
   oc scale machineset <machineset-name> --replicas=2
   ```

2. The cluster will automatically expand

## Troubleshooting

### Common Issues

#### High Resource Usage

- Monitor with `oc adm top nodes`
- Check running pods: `oc get pods --all-namespaces`
- Scale up resources if needed

#### Network Issues

- Verify network configuration
- Check DNS resolution
- Validate firewall rules

#### Storage Issues

- Check available disk space
- Verify storage class configuration
- Monitor PVC status

### Logs and Debugging

- View cluster logs: `oc logs -n openshift-cluster-version <pod>`
- Check node logs: `journalctl -u kubelet`
- Use `oc debug` for troubleshooting

### Recovery

For severe issues, SNO can self-heal. If needed:

1. Reboot the node
2. The cluster will automatically recover

## Use Cases

### Edge Computing

- Deploy at remote locations
- Low power consumption
- Autonomous operation

### Development and Testing

- Quick setup for development
- Isolated environments
- Cost-effective testing

### Production Workloads

- Small-scale applications
- Proof-of-concept deployments
- Resource-constrained environments

## Limitations

- No high availability (single point of failure)
- Limited scalability
- Reduced fault tolerance
- Not suitable for large-scale production

## Best Practices

- Use SSD storage for better performance
- Configure monitoring and alerting
- Regular backups of important data
- Keep the system updated
- Monitor resource usage continuously