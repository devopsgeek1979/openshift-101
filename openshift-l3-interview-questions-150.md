# OpenShift Interview Bank (150 Questions: Beginner to L3)

This guide provides **150 interview questions** progressing from beginner to L3 depth.
It explicitly covers:

- Linux/OS fundamentals required for OpenShift administration
- Critical OpenShift/Kubernetes ports and connectivity reasoning
- Most common failure reasons and root-cause based troubleshooting

## How to Use

- Questions **1-50**: Beginner fundamentals (platform, Linux, networking, cluster basics)
- Questions **51-100**: Intermediate administration and troubleshooting
- Questions **101-150**: Advanced L3 scenarios, deep diagnostics, architecture, and recovery

---

## Section A — Beginner Foundation (1-50)

1. What are the main control plane components in OpenShift, and what does each do?
2. What is the difference between a Pod, Deployment, ReplicaSet, and StatefulSet?
3. What is the purpose of `kubelet` on a worker node?
4. What is the role of `CRI-O` in OpenShift nodes?
5. What happens from `oc apply` to a pod becoming Running?
6. Which OS families are typically supported for OpenShift nodes, and why?
7. Why is time synchronization (NTP/chrony) critical for OpenShift?
8. What Linux command sequence do you use to check CPU, memory, disk, and load quickly?
9. How do `systemctl`, `journalctl`, and `dmesg` help in node-level troubleshooting?
10. What is SELinux, and why should it remain Enforcing in production?
11. What are Linux cgroups, and how do they relate to pod resource limits?
12. What is a Linux namespace, and which namespaces are used by containers?
13. Which port does the Kubernetes/OpenShift API server use, and why is it critical?
14. Which port is used for etcd client traffic in control plane communication?
15. Which ports are required for machine config updates and SSH-based node checks?
16. How do you verify if a specific port is open from one node to another?
17. What happens if DNS is misconfigured for `api.<cluster-domain>`?
18. What is CoreDNS in OpenShift, and how do you validate DNS health?
19. Why do pods fail with `ImagePullBackOff`, and what are the top 5 checks?
20. What is the function of an ImagePullSecret, and where is it configured?
21. How do you inspect pod startup failures using `oc describe pod`?
22. Why does a pod enter `CrashLoopBackOff`, and how do you isolate application vs platform cause?
23. What is the difference between liveness, readiness, and startup probes?
24. How can incorrect readiness probes cause a production outage?
25. What is a Service in Kubernetes/OpenShift, and how does ClusterIP routing work?
26. What is an OpenShift Route, and how is it different from a Kubernetes Ingress?
27. What is the role of the ingress controller in OpenShift?
28. Which port is usually used for HTTPS routes, and what TLS termination modes exist?
29. What are common reasons for `503` on an OpenShift Route?
30. How do you check whether router pods are healthy and scheduling correctly?
31. What is a namespace/project in OpenShift, and why is it important for isolation?
32. What are ResourceQuota and LimitRange, and how can they break deployments?
33. What is an SCC (Security Context Constraint), and why do pods fail with SCC errors?
34. What is the default SCC behavior for non-privileged workloads?
35. Why are hostPath volumes restricted in most production environments?
36. What is persistent storage in OpenShift, and when do you choose block vs file?
37. What are PV and PVC, and how does binding happen?
38. Why does a PVC remain Pending, and what are the first checks?
39. What are common storage failure indicators in pod events?
40. How do you identify node pressure conditions (MemoryPressure, DiskPressure, PIDPressure)?
41. What happens when ephemeral storage is exhausted on a node?
42. Which logs are most useful when node NotReady occurs?
43. What is cluster operator status, and which states require immediate action?
44. What is `oc get clusterversion`, and why is it central during upgrades?
45. Why do certificate issues commonly break API or ingress access?
46. What Linux file permissions problems can break kubelet or CRI-O runtime behavior?
47. What are common beginner mistakes during install-config creation?
48. Why are IAM/permissions a frequent root cause for cloud IPI failures?
49. What are the most common beginner-level failure reasons in OpenShift day-1 operations?
50. How do you build a minimum troubleshooting checklist before escalating to L2/L3?

---

## Section B — Intermediate Administration & Troubleshooting (51-100)

- Q51. Explain OpenShift Machine API and how it differs from static node management.
- Q52. What are MachineSets, and why can scaling fail despite cloud quota availability?
- Q53. How does autoscaler decide node scale-up, and what blocks it?
- Q54. Which conditions in pending pods prevent Cluster Autoscaler action?
- Q55. How do taints and tolerations cause hidden scheduling failures?
- Q56. What are node selectors and affinity rules, and how can they deadlock scheduling?
- Q57. How do PodDisruptionBudgets affect upgrades and node drains?
- Q58. Why can draining a node fail repeatedly during maintenance windows?
- Q59. What is the safest sequence for worker node replacement?
- Q60. Which OS-level checks do you run before admitting a new worker node?
- Q61. How do MTU mismatches impact pod-to-pod traffic in OpenShift SDN/OVN-K?
- Q62. What are common OVN-Kubernetes failure symptoms and first diagnostics?
- Q63. Which ports must be reachable between control plane and workers for stable operation?
- Q64. How do you validate east-west traffic between namespaces with network policies enabled?
- Q65. How can an overly strict NetworkPolicy break DNS and metrics silently?
- Q66. What is the fastest way to identify dropped traffic due to policy misconfiguration?
- Q67. Why does API server latency spike under etcd stress?
- Q68. How do you check etcd health, member status, and defragmentation need?
- Q69. What are warning signs of etcd disk I/O bottlenecks?
- Q70. Why does etcd clock skew produce cluster instability?
- Q71. What is the impact of failed certificate rotation in kube-apiserver or ingress?
- Q72. How do you safely rotate certificates and validate post-rotation health?
- Q73. Which OpenShift operators are most critical to monitor continuously and why?
- Q74. How do degraded authentication operators affect cluster access patterns?
- Q75. What are common LDAP/OIDC integration failures and quick rollback options?
- Q76. How do you troubleshoot OAuth login loops and callback URL mismatches?
- Q77. What causes image registry push/pull failures in internal registry setups?
- Q78. How do you recover from registry storage backend latency or outage?
- Q79. What is the role of image pruner, and how can bad pruning break workloads?
- Q80. Why do builds fail intermittently when node disk usage is high?
- Q81. What is the difference between BuildConfig failure and deployment failure?
- Q82. How do you debug failing init containers in CI/CD workloads?
- Q83. Why does a rollout hang at `Progressing=True` but never complete?
- Q84. How do you detect readiness probe flapping due to downstream dependency latency?
- Q85. What causes random pod OOMKills even with apparently sufficient limits?
- Q86. How do Linux memory overcommit and cgroup limits interact in containerized apps?
- Q87. What are top causes of high context switching and CPU throttling in pods?
- Q88. How can `ulimit` or PID limits trigger hidden app instability?
- Q89. What are common causes of node filesystem inode exhaustion?
- Q90. How do you diagnose log growth causing node disk pressure?
- Q91. What is the impact of failed MachineConfig rollout on node pools?
- Q92. How do you recover when a node is stuck in `Updating` from MCO?
- Q93. How do you triage upgrade stalls at 40-70% completion?
- Q94. What pre-upgrade validations prevent most OpenShift upgrade failures?
- Q95. Why is disconnected cluster upgrade often failing at catalog/operator sync?
- Q96. Which ports and proxies must be validated for disconnected environments?
- Q97. How do you troubleshoot CSI driver failures for attach/mount operations?
- Q98. What are recurring Ceph/NFS storage failure reasons in production clusters?
- Q99. How do you build an intermediate-level incident timeline for postmortems?
- Q100. What are the top 10 intermediate failure patterns you automate alerts for?

---

## Section C — Advanced L3 Deep-Dive (101-150)

- Q101. Design a full L3 triage workflow for API outage with partial worker health.
- Q102. How do you isolate control plane failure from external load balancer failure?
- Q103. What are advanced diagnostics for intermittent API server `EOF` and timeout errors?
- Q104. How do you correlate API latency with etcd fsync and network RTT metrics?
- Q105. What is your L3 playbook for repeated etcd leader elections under peak load?
- Q106. How do you decide between etcd defrag, member replacement, or infra scaling?
- Q107. Which OS kernel parameters commonly impact high-scale OpenShift control planes?
- Q108. How do you tune `sysctl` safely for throughput without destabilizing workloads?
- Q109. What are kernel-level causes of dropped packets affecting pod networking?
- Q110. How do conntrack exhaustion and NAT table limits manifest in OpenShift traffic?
- Q111. Which ports are mandatory for OVN, kubelet, API, ingress, monitoring, and logging paths?
- Q112. How do you perform end-to-end port path validation for north-south traffic?
- Q113. How do you debug TLS handshake failures between router and backend service pods?
- Q114. What causes route admitted status true but persistent 503 responses?
- Q115. How do you diagnose mTLS and SNI issues in service mesh + OpenShift route chains?
- Q116. How do you implement blue-green with strict rollback RTO under 2 minutes?
- Q117. How do you validate data consistency during blue-green cutover for stateful services?
- Q118. What is your strategy for dual-write and schema compatibility in zero-downtime DB migration?
- Q119. How do you instrument canary analysis with SLO-based promotion gates?
- Q120. Which failure signals should auto-trigger rollback in GitOps pipelines?
- Q121. How do you prevent ArgoCD drift in clusters with manual emergency changes?
- Q122. What are top ArgoCD sync failure root causes at enterprise scale?
- Q123. How do you shard ArgoCD controllers for 500+ apps and multi-cluster targets?
- Q124. How do you secure ArgoCD repo credentials, tokens, and signing verification?
- Q125. What are the failure modes of ApplicationSet generators in dynamic cluster fleets?
- Q126. How do you design RBAC boundaries across platform team, app team, and security team?
- Q127. How do SCC, PSA, and admission webhooks interact in modern OpenShift hardening?
- Q128. What causes admission webhook latency to impact cluster-wide deployment throughput?
- Q129. How do you diagnose webhook deadlocks and fail-open vs fail-close risk?
- Q130. How do you perform forensic analysis after suspicious lateral movement across namespaces?
- Q131. What logs and audit events are mandatory for compliance-grade incident reconstruction?
- Q132. How do you design immutable evidence collection in OpenShift security incidents?
- Q133. What are top causes of control plane CPU saturation and remediation sequence?
- Q134. How do you determine if bottleneck is app, platform, network, or storage under pressure?
- Q135. What are advanced causes of kubelet instability on specific kernel/runtime versions?
- Q136. How do you recover from widespread node NotReady caused by bad MachineConfig push?
- Q137. How do you execute safe rollback when all worker pools are partially degraded?
- Q138. What is your plan for region-level cloud outage impacting OpenShift IPI components?
- Q139. How do you architect multi-region failover for low RPO and low RTO?
- Q140. How do you validate disaster recovery readiness beyond backup success messages?
- Q141. What are common reasons backups are unusable during real restore events?
- Q142. How do you validate etcd snapshots for actual restorability and consistency?
- Q143. What is your L3 approach for intermittent storage latency spikes causing app timeouts?
- Q144. How do you distinguish CSI control plane issue vs backend array/Ceph issue quickly?
- Q145. Which OS-level metrics best predict node failure before Kubernetes surfaces conditions?
- Q146. How do you build predictive alerting to catch recurring failure reasons early?
- Q147. How do you standardize root-cause taxonomy for OpenShift incidents across teams?
- Q148. How do you convert top recurring incidents into self-healing automation runbooks?
- Q149. How do you evaluate platform maturity from beginner operations to L3 excellence?
- Q150. If given a failing production cluster, what exact first 30 minutes L3 actions do you execute?

---

## Coverage Map

### OS Depth Check Areas

- Linux boot/systemd/journal diagnostics
- cgroups, namespaces, SELinux, sysctl, ulimit, PID/memory/inode pressure
- kubelet/CRI-O runtime interactions and kernel/network stack behavior

### Port Knowledge Check Areas

- API server, etcd, kubelet, ingress/router, DNS, monitoring/logging and node-to-node paths
- Port/path validation across control plane, worker, and external clients
- TLS/mTLS/SNI and proxy/firewall dependency checks

### Common Failure Reasons Emphasized

- DNS, certificates, IAM/permissions, quota exhaustion
- Network policy, MTU/conntrack, probe misconfiguration
- Storage attach/mount latency and backend instability
- Upgrade stalls, operator degradation, MachineConfig rollout failure
- GitOps drift/sync failures, blue-green cutover and rollback errors

## Suggested Scoring Rubric (Optional)

- Beginner (Q1-50): 1 point each
- Intermediate (Q51-100): 2 points each
- Advanced L3 (Q101-150): 3 points each

Interpretation:

- 0-120: Needs foundational strengthening
- 121-220: Operationally capable, limited L3 depth
- 221-320: Strong L3 readiness
- 321-400: Senior L3 / Lead-level troubleshooting maturity
