# ✨ OpenShift MicroShift Installation

MicroShift is a lightweight, Kubernetes-optimized distribution of OpenShift designed for edge computing and IoT scenarios. It provides a minimal OpenShift experience with reduced resource requirements.

## 🔹 Key Features

- Lightweight Kubernetes distribution
- Optimized for edge devices
- Low resource footprint
- Over-the-air (OTA) updates
- Built-in monitoring and logging
- Security-focused design

## 🔹 Prerequisites

- RHEL 8.6+ or RHEL 9.x
- Minimum: 2 CPU cores, 2 GB RAM, 10 GB storage
- Red Hat subscription
- Network connectivity

## 🔹 Installation Methods

### 📌 Method 1: RPM Installation

1. Register the system:

   ```bash
   subscription-manager register
   subscription-manager attach --auto
   ```

2. Enable required repositories:

   ```bash
   subscription-manager repos --enable=rhel-8-for-x86_64-appstream-rpms
   subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms
   ```

3. Install MicroShift:

   ```bash
   dnf install -y microshift
   ```

4. Start MicroShift:

   ```bash
   systemctl enable microshift --now
   ```

### 📌 Method 2: Container Installation

For containerized deployment:

```bash
podman run -d --name microshift \
  -p 6443:6443 \
  -v /var/lib/microshift:/var/lib/microshift \
  registry.redhat.io/microshift/microshift:latest
```

## 🔹 Post-Installation Setup

### 📌 Configure Firewall

```bash
firewall-cmd --permanent --zone=public --add-port=6443/tcp
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-port=443/tcp
firewall-cmd --reload
```

### 📌 Access the Cluster

1. Get kubeconfig:

   ```bash
   mkdir -p ~/.kube
   cp /var/lib/microshift/resources/kubeadmin/kubeconfig ~/.kube/config
   ```

2. Access the API:

   ```bash
   export KUBECONFIG=/var/lib/microshift/resources/kubeadmin/kubeconfig
   oc login -u kubeadmin -p $(cat /var/lib/microshift/resources/kubeadmin/password)
   ```

### 📌 Install Add-ons

MicroShift supports various add-ons:

```bash
# Install console
oc apply -f https://raw.githubusercontent.com/openshift/microshift/main/deploy/addons/console.yaml

# Install monitoring
oc apply -f https://raw.githubusercontent.com/openshift/microshift/main/deploy/addons/monitoring.yaml
```

## 🔹 Configuration

### 📌 Custom Configuration

Edit `/etc/microshift/config.yaml`:

```yaml
apiServer:
  subjectAltNames:
    - microshift.example.com
network:
  clusterNetwork:
    - cidr: 10.42.0.0/16
  serviceNetwork:
    - 10.43.0.0/16
storage:
  driver: csi-hostpath
```

### 📌 Network Configuration

MicroShift uses flannel by default. For custom networking:

```yaml
network:
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  serviceNetwork:
    - 172.30.0.0/16
```

## 🔹 Management

### 📌 Updates

MicroShift supports OTA updates:

```bash
microshift-update
```

### 📌 Monitoring

Access built-in monitoring:

```bash
oc get routes -n microshift-monitoring
```

### 📌 Logs

View MicroShift logs:

```bash
journalctl -u microshift
```

## 🔹 Troubleshooting

### 📌 Service Not Starting

Check status:

```bash
systemctl status microshift
journalctl -u microshift -f
```

### 📌 API Server Issues

Verify configuration:

```bash
microshift show-config
```

### 📌 Network Problems

Check network configuration:

```bash
oc get network
oc describe network
```

### 📌 Storage Issues

Verify storage setup:

```bash
oc get storageclass
oc get pv
```

## 🔹 Security

### 📌 Certificate Management

MicroShift automatically manages certificates. To view:

```bash
oc get secrets -n microshift-system
```

### 📌 User Authentication

Configure authentication providers as needed.

## 🔹 Performance Tuning

### 📌 Resource Limits

Adjust resource limits in `/etc/microshift/config.yaml`:

```yaml
apiServer:
  resources:
    requests:
      cpu: 500m
      memory: 1Gi
    limits:
      cpu: 1
      memory: 2Gi
```

### 📌 Storage Optimization

Use efficient storage drivers for edge devices.

## 🔹 Use Cases

### 📌 Edge Computing

- IoT gateways
- Remote monitoring
- Field devices

### 📌 Development

- Local development environments
- CI/CD pipelines
- Testing platforms

### 📌 Embedded Systems

- Industrial control systems
- Network appliances
- Embedded applications

## 🔹 Limitations

- No high availability
- Limited scalability
- Reduced feature set compared to full OpenShift
- Manual scaling required

## 🔹 Best Practices

- Regular updates and patches
- Monitor resource usage
- Implement proper backup strategies
- Use network policies for security
- Configure logging and monitoring
