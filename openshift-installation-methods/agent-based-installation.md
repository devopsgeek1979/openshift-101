# OpenShift Agent-Based Installer

The Agent-based installer provides a streamlined installation experience for OpenShift Container Platform. It uses a bootable ISO image to install the cluster, making it suitable for disconnected environments and air-gapped installations.

## Key Features

- Single ISO image for all nodes
- Supports disconnected installations
- Simplified configuration
- Integrated with Assisted Installer service

## Prerequisites

- RHEL 8.6 or later servers
- Minimum 3 master nodes and 2 worker nodes
- 8 CPU cores, 16 GB RAM per node
- 120 GB storage per node
- Network connectivity between nodes
- DHCP server (optional, can use static IPs)

## Installation Steps

### 1. Download the Agent-Based Installer

```bash
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
tar -xzf openshift-install-linux.tar.gz
```

### 2. Create Agent Config

Generate the agent-config.yaml:

```bash
./openshift-install agent create config --dir=.
```

Or create manually:

```yaml
apiVersion: v1beta1
kind: AgentConfig
metadata:
  name: my-cluster
rendezvousIP: 192.168.1.10  # IP of bootstrap node
hosts:
  - hostname: master-1
    interfaces:
      - name: enp1s0
        macAddress: 00:00:00:00:00:01
    networkConfig:
      interfaces:
        - name: enp1s0
          type: ethernet
          state: up
          mac-address: 00:00:00:00:00:01
          ipv4:
            enabled: true
            address:
              - ip: 192.168.1.11
                prefix-length: 24
            dhcp: false
  - hostname: master-2
    interfaces:
      - name: enp1s0
        macAddress: 00:00:00:00:00:02
    networkConfig:
      interfaces:
        - name: enp1s0
          type: ethernet
          state: up
          mac-address: 00:00:00:00:00:02
          ipv4:
            enabled: true
            address:
              - ip: 192.168.1.12
                prefix-length: 24
            dhcp: false
  - hostname: master-3
    interfaces:
      - name: enp1s0
        macAddress: 00:00:00:00:00:03
    networkConfig:
      interfaces:
        - name: enp1s0
          type: ethernet
          state: up
          mac-address: 00:00:00:00:00:03
          ipv4:
            enabled: true
            address:
              - ip: 192.168.1.13
                prefix-length: 24
            dhcp: false
```

### 3. Create PXE Assets (Optional)

For PXE booting:

```bash
./openshift-install agent create pxe --dir=.
```

This generates:

- PXE boot files
- initrd and kernel images
- ignition configs

### 4. Create ISO Image

```bash
./openshift-install agent create image --dir=.
```

### 5. Boot Nodes

- Boot each node with the generated ISO or PXE
- The agent will automatically discover other nodes
- Nodes will form the cluster without manual intervention

### 6. Monitor Installation

```bash
./openshift-install agent wait-for bootstrap-complete --dir=.
./openshift-install agent wait-for install-complete --dir=.
```

### 7. Access Cluster

```bash
export KUBECONFIG=auth/kubeconfig
oc login -u kubeadmin
```

## Network Configuration

### Static IP Configuration

Use NMState for advanced networking:

```yaml
networkConfig:
  interfaces:
    - name: enp1s0
      type: ethernet
      state: up
      ipv4:
        enabled: true
        address:
          - ip: 192.168.1.10
            prefix-length: 24
        dhcp: false
  dns-resolver:
    config:
      server:
        - 8.8.8.8
  routes:
    config:
      - destination: 0.0.0.0/0
        next-hop-address: 192.168.1.1
        next-hop-interface: enp1s0
```

### DHCP Configuration

For DHCP environments, minimal config:

```yaml
networkConfig:
  interfaces:
    - name: enp1s0
      type: ethernet
      state: up
      ipv4:
        enabled: true
        dhcp: true
```

## Disconnected Installation

For air-gapped environments:

1. Mirror required images to local registry
2. Configure install-config.yaml with local registry
3. Use local DNS and NTP servers
4. Ensure all nodes can reach local registry

## Troubleshooting

### Node Discovery Issues

- Verify network connectivity between nodes
- Check firewall rules
- Ensure rendezvousIP is accessible

### Installation Failures

- Check agent logs: `journalctl -u agent.service`
- Verify hardware requirements
- Validate network configuration

### Bootstrap Issues

- Check bootstrap logs
- Verify ignition configs
- Ensure proper DNS resolution

## Failure Scenarios and Troubleshooting

### Scenario 1: Nodes Not Discovered

**Symptoms:**

- Nodes don't appear in the cluster
- Agent service fails to start

**Possible Causes:**

- Incorrect rendezvousIP
- Network connectivity issues
- Firewall blocking agent communication

**Troubleshooting Steps:**

1. Check agent logs:

   ```bash
   journalctl -u agent.service -f
   ```

2. Verify network configuration:

   ```bash
   nmcli connection show
   ip route show
   ```

3. Test connectivity to rendezvous IP:

   ```bash
   ping <rendezvous-ip>
   nc -zv <rendezvous-ip> 6443
   ```

**Resolution:**

- Correct rendezvousIP in agent-config.yaml
- Ensure all nodes can reach the bootstrap node
- Check firewall rules

### Scenario 2: Installation Hangs at Bootstrap

**Symptoms:**

- Bootstrap process doesn't complete
- Nodes show "Installing" status indefinitely

**Possible Causes:**

- Insufficient resources
- Network timeouts
- Invalid configuration

**Troubleshooting Steps:**

1. Monitor installation progress:

   ```bash
   openshift-install agent wait-for bootstrap-complete --log-level debug --dir=.
   ```

2. Check resource usage:

   ```bash
   top
   free -h
   ```

3. Validate agent config:

   ```bash
   cat agent-config.yaml
   ```

**Resolution:**

- Increase node resources
- Check network stability
- Regenerate agent config

### Scenario 3: Certificate Issues

**Symptoms:**

- API server certificate errors
- Nodes fail authentication

**Possible Causes:**

- Clock synchronization issues
- Invalid certificates
- DNS problems

**Troubleshooting Steps:**

1. Check system time:

   ```bash
   timedatectl
   ```

2. Verify certificates:

   ```bash
   oc get secrets -n openshift-kube-apiserver
   ```

3. Test DNS resolution:

   ```bash
   nslookup api.cluster.example.com
   ```

**Resolution:**

- Synchronize clocks with NTP
- Regenerate cluster certificates
- Fix DNS configuration

### Common Troubleshooting Questions

1. **Q: Agent service fails to start on nodes?**
   A: Check ignition config and ensure proper disk partitioning.

2. **Q: Nodes can't communicate with rendezvous IP?**
   A: Verify network configuration and firewall rules.

3. **Q: Installation fails with "no available nodes" error?**
   A: Check agent-config.yaml for correct node specifications.

4. **Q: Bootstrap node runs out of disk space?**
   A: Ensure adequate storage for container images and logs.

5. **Q: PXE boot fails on some nodes?**
   A: Verify DHCP server configuration and PXE assets.

## Best Practices for Agent-Based Troubleshooting

- Validate agent-config.yaml before booting nodes
- Monitor agent logs on all nodes during installation
- Ensure consistent network configuration across nodes
- Keep detailed records of hardware specifications
- Test PXE boot process in staging environment
