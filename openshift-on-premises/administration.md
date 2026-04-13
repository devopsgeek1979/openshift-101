# OpenShift On-Premises Administration Guide

This guide covers day-to-day administration tasks for OpenShift Container Platform on-premises.

## Cluster Management

### Node Management

- Add worker nodes:

  ```bash
  oc scale machineset <machineset-name> --replicas=<count>
  ```

- Drain a node for maintenance:

  ```bash
  oc adm drain <node-name> --ignore-daemonsets --delete-emptydir-data
  ```

- Uncordon a node:

  ```bash
  oc adm uncordon <node-name>
  ```

### User and Authentication

- Create a user:

  ```bash
  oc create user <username>
  ```

- Assign cluster-admin role:

  ```bash
  oc adm policy add-cluster-role-to-user cluster-admin <username>
  ```

- Configure identity providers (LDAP, etc.) via OAuth

### Storage Management

- Create a persistent volume:

  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv001
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: /tmp/data
  ```

- Configure storage classes for different backends (NFS, Ceph, etc.)

## Monitoring and Logging

- Check cluster status:

  ```bash
  oc get clusteroperators
  ```

- View pod logs:

  ```bash
  oc logs <pod-name> -n <namespace>
  ```

- Monitor resource usage:

  ```bash
  oc adm top nodes
  oc adm top pods
  ```

## Backup and Recovery

### ETCD Backup

1. Stop the cluster:

   ```bash
   oc adm wait-for-stable-api
   ```

2. Backup ETCD:

   ```bash
   oc adm etcd snapshot save /path/to/backup.db
   ```

3. Restore:

   ```bash
   oc adm etcd snapshot restore /path/to/backup.db
   ```

### Application Backup

Use OpenShift API for Data Protection (OADP) for application backups.

## Updates and Upgrades

- Check available updates:

  ```bash
  oc adm upgrade
  ```

- Perform an upgrade:

  ```bash
  oc adm upgrade --to=<version>
  ```

- Update channel management via ClusterVersion resource

## Security

- Configure security context constraints (SCCs)
- Manage secrets and config maps
- Set up network policies
- Enable audit logging

## Networking

- Configure ingress controllers
- Set up load balancers
- Manage routes and services
- Troubleshoot network issues with `oc get network` and `oc describe network`
