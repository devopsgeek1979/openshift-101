# OpenShift Installation with Ansible

Ansible can be used to automate OpenShift Container Platform installation, especially for disconnected environments and custom configurations. This guide covers using Ansible playbooks for OpenShift deployment.

## Prerequisites

- Ansible 2.9+
- Python 3.6+
- OpenShift Ansible playbooks (from openshift-ansible repository)
- Target hosts with RHEL/CentOS
- SSH access to all nodes
- Red Hat subscription

## Installation Methods

### Method 1: OpenShift Ansible Playbooks

1. **Install Ansible and dependencies:**

   ```bash
   # On control node
   dnf install -y ansible python3-pip
   pip3 install openshift jmespath

   # Clone openshift-ansible repository
   git clone https://github.com/openshift/openshift-ansible.git
   cd openshift-ansible
   ```

2. **Configure inventory file:**

   Create `/etc/ansible/hosts` or custom inventory:

   ```ini
   [OSEv3:children]
   masters
   nodes
   etcd

   [OSEv3:vars]
   ansible_ssh_user=root
   openshift_deployment_type=openshift-enterprise
   openshift_release=v3.11
   openshift_master_default_subdomain=apps.example.com
   openshift_master_cluster_hostname=master.example.com
   openshift_master_cluster_public_hostname=master.example.com

   [masters]
   master1.example.com

   [nodes]
   master1.example.com openshift_node_group_name='node-config-master'
   node1.example.com openshift_node_group_name='node-config-compute'
   node2.example.com openshift_node_group_name='node-config-compute'

   [etcd]
   master1.example.com
   ```

3. **Configure group variables:**

   Create `group_vars/OSEv3.yml`:

   ```yaml
   ---
   openshift_master_cluster_method: native
   openshift_master_cluster_hostname: master.example.com
   openshift_master_cluster_public_hostname: master.example.com
   openshift_master_default_subdomain: apps.example.com

   # Authentication
   openshift_master_identity_providers:
   - name: htpasswd_auth
     login: true
     challenge: true
     kind: HTPasswdPasswordIdentityProvider
     filename: /etc/origin/master/htpasswd

   # Registry
   openshift_hosted_registry_storage_kind: nfs
   openshift_hosted_registry_storage_access_modes: ['ReadWriteMany']
   openshift_hosted_registry_storage_nfs_directory: /exports
   openshift_hosted_registry_storage_nfs_options: '*(rw,root_squash)'
   openshift_hosted_registry_storage_volume_name: registry
   openshift_hosted_registry_storage_volume_size: 100Gi

   # Metrics
   openshift_metrics_install_metrics: true
   openshift_metrics_storage_kind: nfs
   openshift_metrics_storage_access_modes: ['ReadWriteOnce']
   openshift_metrics_storage_nfs_directory: /exports
   openshift_metrics_storage_nfs_options: '*(rw,root_squash)'
   openshift_metrics_storage_volume_name: metrics
   openshift_metrics_storage_volume_size: 20Gi

   # Logging
   openshift_logging_install_logging: true
   openshift_logging_storage_kind: nfs
   openshift_logging_storage_access_modes: ['ReadWriteOnce']
   openshift_logging_storage_nfs_directory: /exports
   openshift_logging_storage_nfs_options: '*(rw,root_squash)'
   openshift_logging_storage_volume_name: logging
   openshift_logging_storage_volume_size: 20Gi
   ```

4. **Run prerequisites playbook:**

   ```bash
   ansible-playbook -i inventory.ini playbooks/prerequisites.yml
   ```

5. **Deploy OpenShift:**

   ```bash
   ansible-playbook -i inventory.ini playbooks/deploy_cluster.yml
   ```

### Method 2: Custom Ansible Playbooks for OCP 4

For OpenShift 4, create custom playbooks for post-installation configuration:

```yaml
---
- name: Configure OpenShift Cluster
  hosts: localhost
  connection: local
  gather_facts: false

  tasks:
  - name: Login to cluster
    command: oc login {{ openshift_api_url }} --token={{ openshift_token }}
    register: login_result

  - name: Create project
    command: oc new-project {{ project_name }}
    register: project_result

  - name: Deploy application
    command: oc apply -f {{ application_manifest }}
    register: deploy_result

  - name: Configure network policies
    command: oc apply -f {{ network_policy_manifest }}
    register: network_result

  - name: Setup monitoring
    command: oc apply -f {{ monitoring_manifest }}
    register: monitoring_result
```

## Advanced Ansible Configurations

### Disconnected Installation

For air-gapped environments:

1. **Mirror images to local registry:**

   ```yaml
   - name: Mirror OpenShift images
     hosts: bastion
     tasks:
     - name: Install podman
       package:
         name: podman
         state: present

     - name: Login to registry
       command: podman login {{ local_registry }} -u {{ registry_user }} -p {{ registry_password }}

     - name: Mirror release images
       command: >
         oc adm release mirror
         --from=quay.io/openshift-release-dev/ocp-release:{{ ocp_version }}-x86_64
         --to={{ local_registry }}/ocp4/openshift4
         --to-release-image={{ local_registry }}/ocp4/openshift4:{{ ocp_version }}-x86_64
   ```

2. **Configure cluster for disconnected environment:**

   ```yaml
   - name: Configure disconnected cluster
     hosts: localhost
     tasks:
     - name: Update image sources
       command: >
         oc patch images.config.openshift.io/cluster
         --type merge -p '{"spec":{"registrySources":{"insecureRegistries":["{{ local_registry }}"]}}}'

     - name: Configure additional trust bundle
       command: >
         oc patch proxy.config.openshift.io/cluster
         --type merge -p '{"spec":{"trustedCA":{"name":"user-ca-bundle"}}}'
   ```

### Multi-Cluster Management

Manage multiple clusters with Ansible:

```yaml
---
- name: Manage Multiple Clusters
  hosts: localhost
  connection: local

  vars:
    clusters:
      - name: prod-cluster
        api_url: https://api.prod.example.com:6443
        token: "{{ prod_token }}"
      - name: dev-cluster
        api_url: https://api.dev.example.com:6443
        token: "{{ dev_token }}"

  tasks:
  - name: Deploy to all clusters
    include_tasks: deploy_app.yml
    loop: "{{ clusters }}"
    loop_control:
      loop_var: cluster

  - name: Check cluster health
    command: oc get nodes --kubeconfig={{ cluster.kubeconfig }}
    register: cluster_status
    loop: "{{ clusters }}"
    ignore_errors: true
```

## Troubleshooting Ansible Deployments

### Common Issues

1. **SSH connectivity failures:**
   - Verify SSH keys and known_hosts
   - Check firewall rules
   - Validate user permissions

2. **Package installation failures:**
   - Check repository configurations
   - Verify Red Hat subscriptions
   - Review package dependencies

3. **Playbook execution errors:**
   - Enable verbose mode: `ansible-playbook -vvv`
   - Check Ansible version compatibility
   - Validate variable definitions

4. **OpenShift deployment failures:**
   - Review OpenShift logs: `oc logs -n openshift-apiserver`
   - Check cluster operator status
   - Validate inventory configuration

### Debugging Techniques

- **Enable debug logging:**

  ```bash
  export ANSIBLE_DEBUG=1
  ansible-playbook -vvv playbook.yml
  ```

- **Check Ansible facts:**

  ```bash
  ansible -m setup hostname
  ```

- **Test connectivity:**

  ```bash
  ansible -m ping all
  ```

- **Run individual tasks:**

  ```bash
  ansible-playbook playbook.yml --step --start-at-task="task name"
  ```

## Best Practices

### Inventory Management

- Use dynamic inventory for cloud environments
- Group hosts logically (masters, workers, etcd)
- Use variables for environment-specific configurations

### Security

- Encrypt sensitive variables with Ansible Vault
- Use SSH key authentication
- Limit playbook execution to specific users

### Idempotency

- Design playbooks to be idempotent
- Use `changed_when` and `failed_when` appropriately
- Test playbooks multiple times

### Version Control

- Store playbooks in Git repositories
- Use branches for different environments
- Document changes and updates

## Integration with CI/CD

### GitLab CI Example

```yaml
stages:
  - deploy

deploy_openshift:
  stage: deploy
  script:
    - ansible-playbook -i inventory.ini deploy.yml
  only:
    - main
  dependencies:
    - build
```

### Jenkins Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy OpenShift') {
            steps {
                ansiblePlaybook(
                    inventory: 'inventory.ini',
                    playbook: 'deploy.yml',
                    credentialsId: 'ansible-ssh-key'
                )
            }
        }
    }
}
```

## Monitoring and Logging

### Ansible Tower/AWX Integration

- Use Ansible Tower for GUI-based execution
- Schedule automated deployments
- Monitor playbook execution history
- Integrate with existing monitoring systems

### Logging Best Practices

- Enable Ansible log collection
- Use callbacks for detailed logging
- Integrate with ELK stack for log analysis
- Archive execution results for auditing
