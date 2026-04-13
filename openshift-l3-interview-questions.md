# OpenShift L3 Administrator Interview Questions

This document contains scenario-based interview questions for L3 OpenShift administrators. These questions test deep technical knowledge, troubleshooting skills, and real-world problem-solving abilities.

## Installation and Deployment Scenarios

### Question 1: Complex IPI Installation Failure

**Scenario:** You're deploying OpenShift 4.12 on AWS using IPI. The bootstrap node starts successfully, but the master nodes fail to join the cluster. The bootstrap logs show "failed to create control plane" errors.

**What would you investigate?**

**Expected Answer:**
1. Check AWS service limits and quotas
2. Verify IAM permissions for the installer
3. Examine VPC/subnet configuration and security groups
4. Check CloudTrail logs for API failures
5. Validate install-config.yaml parameters
6. Review bootstrap ignition config
7. Check network connectivity between bootstrap and masters

**Follow-up:** How would you recover from this situation?

### Question 2: Disconnected Environment Setup

**Scenario:** A customer requires OpenShift deployment in an air-gapped environment with no internet access. They have a local mirror registry but the installation fails during operator installation.

**What steps would you take to troubleshoot and resolve?**

**Expected Answer:**
1. Verify mirror registry connectivity and certificates
2. Check image mirroring process for completeness
3. Validate CatalogSource configurations
4. Ensure Operator Lifecycle Manager (OLM) can access mirrored content
5. Check cluster's additionalTrustBundle configuration
6. Validate imageContentSources in install-config.yaml
7. Test operator installation with custom catalog sources

**Follow-up:** How would you automate the mirroring process for future updates?

### Question 3: Agent-Based Installation Network Issues

**Scenario:** During agent-based installation, some nodes successfully join the cluster while others remain in "Discovering" state. The agent logs show network-related errors.

**What would be your troubleshooting approach?**

**Expected Answer:**
1. Check agent-config.yaml for correct network configuration
2. Verify NMState configurations for each node
3. Test connectivity between nodes and rendezvous IP
4. Check firewall rules and SELinux policies
5. Validate DNS resolution across all nodes
6. Review DHCP server configuration for PXE boot
7. Check for IP address conflicts

**Follow-up:** How would you implement static IP configuration for agent-based installs?

## Cluster Administration Scenarios

### Question 4: Cluster Performance Degradation

**Scenario:** A production OpenShift cluster starts experiencing performance issues. Application response times increase, and some pods are being evicted. The cluster has 50 worker nodes and runs multiple namespaces.

**How would you diagnose and resolve this?**

**Expected Answer:**
1. Check cluster resource usage with `oc adm top nodes` and `oc adm top pods`
2. Review cluster operators status: `oc get clusteroperators`
3. Check etcd performance and database size
4. Analyze network plugin performance
5. Review node conditions and taints
6. Check for resource quotas and limits violations
7. Monitor Prometheus metrics for bottlenecks
8. Review recent changes and deployments

**Follow-up:** What preventive measures would you implement?

### Question 5: Certificate Expiration Crisis

**Scenario:** You're alerted that the OpenShift API server certificates will expire in 24 hours. The cluster is production-critical with zero downtime requirements.

**What's your plan to renew certificates safely?**

**Expected Answer:**
1. Verify current certificate status: `oc get secrets -n openshift-kube-apiserver`
2. Check certificate expiry dates: `oc adm certificates check`
3. Plan certificate rotation during maintenance window
4. Backup current certificates and etcd data
5. Use `oc adm certificates regenerate` for automatic renewal
6. Monitor cluster during rotation
7. Validate all components after renewal
8. Update any external systems with new certificates

**Follow-up:** How would you implement automated certificate monitoring?

### Question 6: Storage Issues at Scale

**Scenario:** A cluster with 100+ PVCs starts experiencing storage issues. Some pods can't mount volumes, and there are I/O timeouts. The storage backend is Ceph RBD.

**What would be your investigation process?**

**Expected Answer:**
1. Check PVC and PV status: `oc get pvc,pv --all-namespaces`
2. Review storage class configurations
3. Check Ceph cluster health: `ceph status`
4. Monitor storage metrics in Prometheus
5. Check network connectivity to Ceph nodes
6. Review CSI driver logs
7. Validate multipath configurations
8. Check for storage quota issues

**Follow-up:** How would you optimize storage performance for high-I/O workloads?

## Networking and Security Scenarios

### Question 7: Network Policy Conflicts

**Scenario:** After implementing strict network policies, several applications lose connectivity. Some services work intermittently, and there are "connection refused" errors in application logs.

**How would you troubleshoot and fix this?**

**Expected Answer:**
1. Review network policy configurations: `oc get networkpolicies --all-namespaces`
2. Check pod selectors and labels
3. Validate service mesh configurations (if applicable)
4. Test connectivity with `oc rsh` and network utilities
5. Review SDN logs: `oc logs -n openshift-sdn <pod>`
6. Check for overlapping or conflicting policies
7. Use `oc adm policy` commands to verify permissions

**Follow-up:** How would you design network policies for microservices architecture?

### Question 8: Authentication Provider Failure

**Scenario:** The cluster's LDAP authentication provider goes down during business hours. Users can't log in, and existing sessions start failing.

**What's your immediate response and long-term solution?**

**Expected Answer:**
1. Check OAuth cluster operator status: `oc get clusteroperators authentication`
2. Review authentication logs: `oc logs -n openshift-authentication <pod>`
3. Verify LDAP server connectivity
4. Check for certificate issues with LDAP
5. Implement temporary local authentication if needed
6. Configure backup authentication provider
7. Set up monitoring for authentication services
8. Implement high availability for auth providers

**Follow-up:** How would you implement multi-provider authentication with failover?

### Question 9: Security Incident Response

**Scenario:** Security monitoring detects suspicious activity - a pod is attempting to access sensitive data outside its namespace. The cluster runs financial applications with strict compliance requirements.

**What steps would you take?**

**Expected Answer:**
1. Isolate the affected pod: `oc delete pod <pod-name>`
2. Check pod security context and SCC
3. Review RBAC permissions for the service account
4. Audit recent changes and deployments
5. Check network policies for unauthorized access
6. Review container images for vulnerabilities
7. Implement additional security controls
8. Document incident for compliance reporting

**Follow-up:** How would you prevent similar incidents in the future?

## Upgrade and Maintenance Scenarios

### Question 10: Failed Cluster Upgrade

**Scenario:** During an OpenShift upgrade from 4.11 to 4.12, the upgrade stalls at 50% completion. Several operators show "UpgradePending" status.

**How would you handle this situation?**

**Expected Answer:**
1. Check clusterversion status: `oc get clusterversion`
2. Review upgrade logs: `oc logs -n openshift-cluster-version <pod>`
3. Check operator statuses: `oc get clusteroperators`
4. Identify failing components
5. Check for resource constraints during upgrade
6. Review upgrade channel and version
7. Consider rollback procedures
8. Check for known issues in errata

**Follow-up:** What pre-upgrade checks would you perform?

### Question 11: ETCD Performance Issues

**Scenario:** ETCD on a large cluster (200+ nodes) shows high latency and frequent leader elections. The cluster becomes unresponsive during peak hours.

**What would be your optimization strategy?**

**Expected Answer:**
1. Check ETCD cluster health: `oc get pods -n openshift-etcd`
2. Monitor ETCD metrics: `etcdctl endpoint health`
3. Review ETCD database size and fragmentation
4. Check network latency between etcd members
5. Optimize etcd configuration parameters
6. Consider etcd defragmentation
7. Review resource allocation for etcd pods
8. Plan for etcd cluster scaling

**Follow-up:** How would you monitor ETCD health proactively?

### Question 12: Disaster Recovery Testing

**Scenario:** You need to test disaster recovery procedures for a critical OpenShift cluster. The test must not impact production operations.

**How would you design and execute this test?**

**Expected Answer:**
1. Create isolated test environment
2. Document all backup procedures
3. Test etcd backup and restore
4. Validate application backup/restore procedures
5. Test cluster recovery from infrastructure failure
6. Verify DNS and load balancer failover
7. Test certificate and secret recovery
8. Document recovery time objectives (RTO) and recovery point objectives (RPO)

**Follow-up:** What automation would you implement for regular DR testing?

## Advanced Troubleshooting Scenarios

### Question 13: Intermittent Pod Failures

**Scenario:** Pods in a specific namespace fail intermittently with "CrashLoopBackOff" status. The failures seem random and don't correlate with resource usage.

**What would be your systematic troubleshooting approach?**

**Expected Answer:**
1. Check pod events: `oc describe pod <pod-name>`
2. Review application logs: `oc logs <pod-name> --previous`
3. Check node conditions and resource availability
4. Review network policies and service mesh
5. Check for image pull issues or registry problems
6. Monitor for OOM kills or other system events
7. Check for anti-affinity rule violations
8. Review security context constraints

**Follow-up:** How would you implement automated pod health monitoring?

### Question 14: Custom Operator Issues

**Scenario:** A custom operator you developed fails to reconcile resources. The operator pod shows "Error" status, and custom resources remain in "Pending" state.

**How would you debug this?**

**Expected Answer:**
1. Check operator logs: `oc logs <operator-pod>`
2. Review custom resource definitions
3. Validate RBAC permissions for the operator
4. Check webhook configurations
5. Review operator SDK logs
6. Test operator logic in isolation
7. Check for resource conflicts
8. Review finalizers and owner references

**Follow-up:** What best practices would you implement for operator development?

### Question 15: Multi-Cluster Management

**Scenario:** You manage 10+ OpenShift clusters across different environments. A security vulnerability requires patching all clusters within 24 hours.

**How would you coordinate this update?**

**Expected Answer:**
1. Assess vulnerability impact across clusters
2. Create update plan with rollback procedures
3. Use OpenShift Cluster Manager for bulk operations
4. Implement staged rollout (dev -> staging -> prod)
5. Monitor cluster health during updates
6. Coordinate with application teams
7. Document changes and validate functionality
8. Implement automated compliance checking

**Follow-up:** How would you implement centralized cluster management?

## Performance and Scaling Scenarios

### Question 16: Cluster Autoscaling Issues

**Scenario:** Cluster autoscaler isn't scaling up nodes despite pending pods. The cluster has sufficient cloud provider quotas.

**What would you investigate?**

**Expected Answer:**
1. Check cluster autoscaler logs: `oc logs -n openshift-machine-api <autoscaler-pod>`
2. Verify MachineSet configurations
3. Check node selectors and taints
4. Review pod disruption budgets
5. Validate cloud provider API permissions
6. Check for resource constraints on existing nodes
7. Review autoscaler configuration parameters

**Follow-up:** How would you optimize autoscaling for cost efficiency?

### Question 17: Application Performance at Scale

**Scenario:** A web application performs well with 100 users but degrades significantly with 1000 concurrent users. The application runs on OpenShift with 20 pods.

**What performance analysis would you conduct?**

**Expected Answer:**
1. Profile application performance with APM tools
2. Check pod resource usage and limits
3. Review service mesh configuration
4. Analyze database connection pooling
5. Check for network bottlenecks
6. Review HPA configurations
7. Monitor JVM/application metrics
8. Check for memory leaks or GC issues

**Follow-up:** How would you implement auto-scaling for this application?

## Final Assessment Questions

### Question 18: Architecture Decision

**Scenario:** A customer wants to deploy OpenShift in a highly regulated environment with strict security requirements, zero trust networking, and automated compliance checking.

**What architecture would you recommend and why?**

### Question 19: Cost Optimization

**Scenario:** A cluster shows 70% average utilization but costs are exceeding budget. The cluster runs mixed workloads with varying resource requirements.

**How would you optimize costs without impacting performance?**

### Question 20: Future-Proofing

**Scenario:** Planning for a 5-year OpenShift deployment supporting 500+ applications with 1000+ developers.

**What architectural decisions would you make for long-term maintainability and scalability?**

## Interview Tips for Candidates

- Demonstrate systematic troubleshooting approach
- Show knowledge of OpenShift internals
- Explain decisions with technical reasoning
- Discuss real-world experience with similar scenarios
- Highlight automation and monitoring implementations
- Emphasize security and compliance considerations
- Show understanding of cloud-native principles
