# ✨ OpenShift On-Premises Installation Guide

This guide covers the installation of OpenShift Container Platform (OCP) on-premises.

## 🔹 Prerequisites

- Red Hat Enterprise Linux (RHEL) 8.x or 9.x servers
- Minimum hardware requirements:
  - 3 master nodes: 4 vCPU, 16 GB RAM, 120 GB storage
  - 3+ worker nodes: 2 vCPU, 8 GB RAM, 120 GB storage
- Red Hat subscription with OpenShift entitlements
- DNS and load balancer configured

## 🔹 Installation Methods

### 📌 1. Assisted Installer (Recommended)

1. Access the Red Hat Hybrid Cloud Console
2. Create a new cluster
3. Select "On-premises" as the platform
4. Follow the guided steps for infrastructure provisioning

### 📌 2. User-Provisioned Infrastructure (UPI)

1. Prepare your infrastructure:
   - Provision VMs or bare metal servers
   - Configure networking (DNS, load balancer)
   - Set up NTP

2. Download the OpenShift installer:

   ```bash
   wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
   tar -xzf openshift-install-linux.tar.gz
   ```

3. Create an install-config.yaml:

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
     none: {}
   pullSecret: '{"auths": {...}}'
   sshKey: 'ssh-ed25519 AAAA...'
   ```

4. Generate ignition configs:

   ```bash
   ./openshift-install create ignition-configs --dir=.
   ```

5. Boot the nodes with the ignition files

6. Monitor installation:

   ```bash
   ./openshift-install wait-for bootstrap-complete --dir=.
   ./openshift-install wait-for install-complete --dir=.
   ```

## 🔹 Post-Installation

1. Access the cluster:

   ```bash
   export KUBECONFIG=auth/kubeconfig
   oc login -u kubeadmin -p <password>
   ```

2. Verify cluster health:

   ```bash
   oc get nodes
   oc get clusteroperators
   ```

3. Configure additional components (registry, monitoring, etc.)

## 🔹 Troubleshooting Installation Issues

- Check bootstrap logs: `oc logs -f bootstrap`
- Verify network connectivity
- Ensure DNS resolution
- Check firewall rules
