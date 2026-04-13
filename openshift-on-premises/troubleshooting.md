# OpenShift On-Premises Troubleshooting Guide

This guide helps troubleshoot common issues in OpenShift Container Platform on-premises deployments.

## Cluster Not Starting

### Cluster Not Starting Symptoms

- Nodes not joining the cluster
- API server unreachable

### Cluster Not Starting Steps

1. Check node status:

   ```bash
   oc get nodes
   ```

2. Verify kubelet logs on nodes:

   ```bash
   journalctl -u kubelet -f
   ```

3. Check network connectivity between nodes

4. Validate certificates:

   ```bash
   oc get secrets -n openshift-kube-apiserver
   ```

## Pod Failures

### Pod Failure Symptoms

- Pods in CrashLoopBackOff or Pending state

### Pod Failure Steps

1. Describe the pod:

   ```bash
   oc describe pod <pod-name> -n <namespace>
   ```

2. Check pod logs:

   ```bash
   oc logs <pod-name> -n <namespace> --previous
   ```

3. Verify resource limits and requests

4. Check for image pull errors:

   ```bash
   oc get events -n <namespace>
   ```

## Networking Issues

### Networking Issue Symptoms

- Services not accessible
- Pod-to-pod communication failing

### Networking Issue Steps

1. Check network policies:

   ```bash
   oc get networkpolicies -n <namespace>
   ```

2. Verify SDN status:

   ```bash
   oc get clusteroperators network
   ```

3. Test connectivity:

   ```bash
   oc rsh <pod-name> curl <service-url>
   ```

## Storage Problems

### Storage Problem Symptoms

- PVCs stuck in Pending
- Pod mounting failures

### Storage Problem Steps

1. Check PVC status:

   ```bash
   oc get pvc -n <namespace>
   oc describe pvc <pvc-name> -n <namespace>
   ```

2. Verify storage class:

   ```bash
   oc get storageclass
   ```

3. Check storage backend (NFS, Ceph, etc.) connectivity

## Authentication Issues

### Authentication Issue Symptoms

- Unable to login
- Permission denied errors

### Authentication Issue Steps

1. Check OAuth status:

   ```bash
   oc get clusteroperators authentication
   ```

2. Verify identity providers:

   ```bash
   oc get oauth
   ```

3. Check user roles:

   ```bash
   oc adm policy who-can <verb> <resource>
   ```

## Performance Issues

### Performance Issue Symptoms

- High latency
- Resource exhaustion

### Performance Issue Steps

1. Monitor resource usage:

   ```bash
   oc adm top nodes
   oc adm top pods
   ```

2. Check cluster autoscaling:

   ```bash
   oc get machinesets
   ```

3. Analyze metrics with Prometheus/Grafana

## Upgrade Failures

### Upgrade Failure Symptoms

- Upgrade stuck
- Components not updating

### Upgrade Failure Steps

1. Check upgrade status:

   ```bash
   oc get clusterversion
   ```

2. View upgrade logs:

   ```bash
   oc logs -n openshift-cluster-version <pod-name>
   ```

3. Force upgrade if needed:

   ```bash
   oc adm upgrade --force
   ```

## Common Logs to Check

- Master logs: `/var/log/openshift-apiserver/`
- Node logs: `journalctl -u kubelet`
- ETCD logs: `oc logs -n openshift-etcd <etcd-pod>`

## Getting Help

- Red Hat Support: Create a support case
- Community: OpenShift forums, Stack Overflow
- Documentation: [https://docs.openshift.com](https://docs.openshift.com)
