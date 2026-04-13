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

## Cleanup

To remove the cluster:

```bash
./openshift-install agent destroy cluster --dir=.
```
