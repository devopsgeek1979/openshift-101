# OpenShift L3 Interview — Combined Questions and Answers (150)

This file merges all 150 questions and answers in one place, organized by category.

## Section A — Beginner Foundation (Q1-Q50)

### Q1

**Question:** What are the main control plane components in OpenShift, and what does each do?

**Answer:** `kube-apiserver` exposes cluster API, `etcd` stores state, `scheduler` places pods, and `controller-manager` reconciles desired vs actual state.

### Q2

**Question:** What is the difference between a Pod, Deployment, ReplicaSet, and StatefulSet?

**Answer:** Pod runs containers; ReplicaSet keeps pod count; Deployment manages rolling updates for stateless pods; StatefulSet provides stable identity/storage ordering for stateful apps.

### Q3

**Question:** What is the purpose of `kubelet` on a worker node?

**Answer:** `kubelet` registers node, starts/stops pods via CRI, reports node/pod status, and executes probes.

### Q4

**Question:** What is the role of `CRI-O` in OpenShift nodes?

**Answer:** `CRI-O` is the container runtime implementing CRI, pulling images and running containers securely on nodes.

### Q5

**Question:** What happens from `oc apply` to a pod becoming Running?

**Answer:** API validates object, persists to etcd, scheduler assigns node, kubelet pulls image/starts container, probes pass, pod becomes Ready.

### Q6

**Question:** Which OS families are typically supported for OpenShift nodes, and why?

**Answer:** RHCOS is default for OCP because immutable OS + MCO-controlled updates give consistency and supportability.

### Q7

**Question:** Why is time synchronization (NTP/chrony) critical for OpenShift?

**Answer:** Cert validation, etcd consensus, token expiry, and distributed logs rely on accurate time; skew causes auth and control-plane instability.

### Q8

**Question:** What Linux command sequence do you use to check CPU, memory, disk, and load quickly?

**Answer:** Typical quick set: `uptime`, `top`, `free -h`, `df -h`, `iostat -x`, `vmstat 1`, `ss -tulpen`.

### Q9

**Question:** How do `systemctl`, `journalctl`, and `dmesg` help in node-level troubleshooting?

**Answer:** `systemctl` checks services, `journalctl` gives service logs/history, `dmesg` shows kernel/hardware/runtime errors.

### Q10

**Question:** What is SELinux, and why should it remain Enforcing in production?

**Answer:** SELinux enforces MAC isolation; disabling it weakens tenant/container boundaries and increases lateral movement risk.

### Q11

**Question:** What are Linux cgroups, and how do they relate to pod resource limits?

**Answer:** cgroups enforce CPU/memory/io constraints; Kubernetes requests/limits are realized through cgroup quotas and limits.

### Q12

**Question:** What is a Linux namespace, and which namespaces are used by containers?

**Answer:** Containers use pid/net/mnt/ipc/uts/user namespaces for process, network, filesystem, IPC, hostname, and user isolation.

### Q13

**Question:** Which port does the Kubernetes/OpenShift API server use, and why is it critical?

**Answer:** API server is on `6443`; every control-plane and client operation depends on reachability and TLS trust to this port.

### Q14

**Question:** Which port is used for etcd client traffic in control plane communication?

**Answer:** etcd client traffic is on `2379` (peer replication commonly `2380`), and disruption breaks state operations.

### Q15

**Question:** Which ports are required for machine config updates and SSH-based node checks?

**Answer:** Commonly validate SSH `22`, kubelet `10250`, machine-config-server `22623` (bootstrap/control-plane path), and API `6443`.

### Q16

**Question:** How do you verify if a specific port is open from one node to another?

**Answer:** Use `nc -vz <host> <port>`, `telnet`, or `curl -k https://<host>:<port>/healthz` plus firewall/security group checks.

### Q17

**Question:** What happens if DNS is misconfigured for `api.<cluster-domain>`?

**Answer:** Bootstrap/control-plane join fails, API endpoint unresolved, and installers/operators timeout.

### Q18

**Question:** What is CoreDNS in OpenShift, and how do you validate DNS health?

**Answer:** CoreDNS resolves cluster service names; validate with `oc get pods -n openshift-dns` and in-pod `dig/nslookup`.

### Q19

**Question:** Why do pods fail with `ImagePullBackOff`, and what are the top 5 checks?

**Answer:** Top checks: image name/tag, pull secret auth, registry DNS/TLS reachability, network/proxy path, and registry rate limits.

### Q20

**Question:** What is the function of an ImagePullSecret, and where is it configured?

**Answer:** ImagePullSecret stores registry credentials and is attached at service account/pod level for private image pulls.

### Q21

**Question:** How do you inspect pod startup failures using `oc describe pod`?

**Answer:** `oc describe pod` shows events (image pull/probe/scheduling errors), conditions, and reason transitions.

### Q22

**Question:** Why does a pod enter `CrashLoopBackOff`, and how do you isolate application vs platform cause?

**Answer:** CrashLoopBackOff is repeated start-fail cycles; compare container logs/app config with platform events/resources/probes.

### Q23

**Question:** What is the difference between liveness, readiness, and startup probes?

**Answer:** Liveness restarts dead app, readiness controls traffic eligibility, startup delays liveness until app bootstrap completes.

### Q24

**Question:** How can incorrect readiness probes cause a production outage?

**Answer:** Failing readiness removes healthy pods from service endpoints, causing partial/total outage despite running containers.

### Q25

**Question:** What is a Service in Kubernetes/OpenShift, and how does ClusterIP routing work?

**Answer:** Service gives stable virtual IP/DNS and load-balances to backend endpoints selected by labels.

### Q26

**Question:** What is an OpenShift Route, and how is it different from a Kubernetes Ingress?

**Answer:** Route is OpenShift-native HTTP(S) exposure with router integration; Ingress is Kubernetes API abstraction.

### Q27

**Question:** What is the role of the ingress controller in OpenShift?

**Answer:** Ingress controller/router terminates TLS (as configured), routes external traffic to in-cluster services.

### Q28

**Question:** Which port is usually used for HTTPS routes, and what TLS termination modes exist?

**Answer:** HTTPS commonly on `443`; termination modes are edge, reencrypt, and passthrough.

### Q29

**Question:** What are common reasons for `503` on an OpenShift Route?

**Answer:** No ready endpoints, wrong target port, failed backend probes, router issues, policy/firewall blocks, or TLS mismatch.

### Q30

**Question:** How do you check whether router pods are healthy and scheduling correctly?

**Answer:** Check router deployment/pods/events/logs in `openshift-ingress`, plus route admission and endpoint health.

### Q31

**Question:** What is a namespace/project in OpenShift, and why is it important for isolation?

**Answer:** Namespace provides multitenancy boundaries for RBAC, quotas, policies, and lifecycle isolation.

### Q32

**Question:** What are ResourceQuota and LimitRange, and how can they break deployments?

**Answer:** Quota caps total usage, LimitRange sets default/min/max; mis-sizing blocks pod creation or causes runtime throttling.

### Q33

**Question:** What is an SCC (Security Context Constraint), and why do pods fail with SCC errors?

**Answer:** SCC defines allowed security contexts; pods fail if requested UID/capabilities/host access violate SCC policy.

### Q34

**Question:** What is the default SCC behavior for non-privileged workloads?

**Answer:** Non-privileged workloads generally use restricted SCC behavior: non-root, limited capabilities, confined host access.

### Q35

**Question:** Why are hostPath volumes restricted in most production environments?

**Answer:** `hostPath` bypasses isolation and can expose host data/kernel surface; use only tightly controlled exceptions.

### Q36

**Question:** What is persistent storage in OpenShift, and when do you choose block vs file?

**Answer:** Block for DB/latency-sensitive workloads; file for shared access patterns (RWX) and simpler multi-pod sharing.

### Q37

**Question:** What are PV and PVC, and how does binding happen?

**Answer:** PV is cluster storage resource, PVC is workload claim; controller binds matching capacity/access class.

### Q38

**Question:** Why does a PVC remain Pending, and what are the first checks?

**Answer:** Pending PVC usually means no matching StorageClass/capacity/access mode or CSI provisioning failure.

### Q39

**Question:** What are common storage failure indicators in pod events?

**Answer:** Mount/attach timeout, multi-attach conflict, permission denied, node publish failure, backend unreachable.

### Q40

**Question:** How do you identify node pressure conditions (MemoryPressure, DiskPressure, PIDPressure)?

**Answer:** `oc describe node` conditions + metrics (`oc adm top nodes`) reveal pressure states and scheduling risk.

### Q41

**Question:** What happens when ephemeral storage is exhausted on a node?

**Answer:** Pods get evicted, image pulls/builds fail, kubelet degrades, and node may become NotReady.

### Q42

**Question:** Which logs are most useful when node NotReady occurs?

**Answer:** Start with kubelet and CRI-O journals, MCD logs, node events, kernel logs, and CNI/OVN pod logs.

### Q43

**Question:** What is cluster operator status, and which states require immediate action?

**Answer:** Cluster operators report health; `Degraded=True` or prolonged `Progressing=True` needs immediate investigation.

### Q44

**Question:** What is `oc get clusterversion`, and why is it central during upgrades?

**Answer:** It shows desired/current version and update progression, central for assessing upgrade state and blockers.

### Q45

**Question:** Why do certificate issues commonly break API or ingress access?

**Answer:** Expired/mismatched certs break TLS trust to API/ingress/oauth and can lock out automation/users.

### Q46

**Question:** What Linux file permissions problems can break kubelet or CRI-O runtime behavior?

**Answer:** Wrong ownership/mode on kubelet, CRI-O, CNI, cert, or kubeconfig files can prevent service startup.

### Q47

**Question:** What are common beginner mistakes during install-config creation?

**Answer:** Frequent mistakes: wrong baseDomain, bad pull secret, invalid SSH key, missing proxy/mirror settings, CIDR overlap.

### Q48

**Question:** Why are IAM/permissions a frequent root cause for cloud IPI failures?

**Answer:** IPI creates cloud infra automatically; missing IAM permissions block load balancers, DNS, instances, or storage creation.

### Q49

**Question:** What are the most common beginner-level failure reasons in OpenShift day-1 operations?

**Answer:** Most common: DNS/certs, auth/permissions, image pulls, quota/resource pressure, storage binding, and probe misconfig.

### Q50

**Question:** How do you build a minimum troubleshooting checklist before escalating to L2/L3?

**Answer:** Build checklist: scope impact, API/operators/nodes/pods/storage/network status, recent changes, logs, rollback options.

## Section B — Intermediate Administration & Troubleshooting (Q51-Q100)

### Q51

**Question:** Explain OpenShift Machine API and how it differs from static node management.

**Answer:** Machine API manages node lifecycle declaratively (create/replace/scale) via cloud providers; static nodes are manual lifecycle.

### Q52

**Question:** What are MachineSets, and why can scaling fail despite cloud quota availability?

**Answer:** Scaling can fail from quota, subnet/IP exhaustion, invalid templates, bootstrap/ignition errors, or cloud API throttling.

### Q53

**Question:** How does autoscaler decide node scale-up, and what blocks it?

**Answer:** Autoscaler adds nodes for unschedulable pods if constraints are satisfiable and a matching scalable machine pool exists.

### Q54

**Question:** Which conditions in pending pods prevent Cluster Autoscaler action?

**Answer:** Hard affinity/selector mismatch, missing tolerations, PVC constraints, PDB constraints, and resource requests too high.

### Q55

**Question:** How do taints and tolerations cause hidden scheduling failures?

**Answer:** Taints repel pods unless tolerated; missing tolerations silently keep pods Pending.

### Q56

**Question:** What are node selectors and affinity rules, and how can they deadlock scheduling?

**Answer:** Overly strict affinity/selectors can create zero valid placement sets causing permanent scheduling failure.

### Q57

**Question:** How do PodDisruptionBudgets affect upgrades and node drains?

**Answer:** PDB limits evictions; during upgrades/drains it can block disruption and stall maintenance.

### Q58

**Question:** Why can draining a node fail repeatedly during maintenance windows?

**Answer:** Causes include PDB blocks, daemonset handling, local storage constraints, stuck finalizers, or API pressure.

### Q59

**Question:** What is the safest sequence for worker node replacement?

**Answer:** Cordon, drain with policy awareness, verify replacement node Ready, uncordon, then repeat gradually.

### Q60

**Question:** Which OS-level checks do you run before admitting a new worker node?

**Answer:** Validate OS version, time sync, DNS, NTP, MTU, required ports, runtime health, and cloud metadata reachability.

### Q61

**Question:** How do MTU mismatches impact pod-to-pod traffic in OpenShift SDN/OVN-K?

**Answer:** MTU mismatch causes fragmentation/packet drops, seen as intermittent latency, timeouts, and connection resets.

### Q62

**Question:** What are common OVN-Kubernetes failure symptoms and first diagnostics?

**Answer:** Symptoms: pod network loss, DNS failures, node-to-node breaks; check OVN pods/logs, SB/NB DB health, and node annotations.

### Q63

**Question:** Which ports must be reachable between control plane and workers for stable operation?

**Answer:** Validate API `6443`, kubelet `10250`, etcd `2379/2380` (control-plane), MCS `22623`, DNS `53`, ingress `80/443`.

### Q64

**Question:** How do you validate east-west traffic between namespaces with network policies enabled?

**Answer:** Use test pods + `curl/nc` across namespaces and inspect applied policies/selectors/ports.

### Q65

**Question:** How can an overly strict NetworkPolicy break DNS and metrics silently?

**Answer:** If DNS egress or metrics endpoints are denied, apps may fail indirectly while pods look healthy.

### Q66

**Question:** What is the fastest way to identify dropped traffic due to policy misconfiguration?

**Answer:** Compare policy intent vs live labels and run targeted connectivity tests from failing source pods.

### Q67

**Question:** Why does API server latency spike under etcd stress?

**Answer:** etcd write/read latency backs API persistence/watch operations, increasing API response times.

### Q68

**Question:** How do you check etcd health, member status, and defragmentation need?

**Answer:** Use etcd endpoint health/status alarms, member list, and compaction/defrag indicators.

### Q69

**Question:** What are warning signs of etcd disk I/O bottlenecks?

**Answer:** High fsync duration, disk queue latency, slow commit times, and API tail latency spikes.

### Q70

**Question:** Why does etcd clock skew produce cluster instability?

**Answer:** Clock skew breaks Raft timing assumptions, triggering elections and transient unavailability.

### Q71

**Question:** What is the impact of failed certificate rotation in kube-apiserver or ingress?

**Answer:** Control-plane cert failures block operator-to-API, router-to-backend trust, and client auth.

### Q72

**Question:** How do you safely rotate certificates and validate post-rotation health?

**Answer:** Rotate in controlled window, monitor operators/API/ingress, verify trust chains and endpoint health post-change.

### Q73

**Question:** Which OpenShift operators are most critical to monitor continuously and why?

**Answer:** Typically etcd, kube-apiserver, authentication, ingress, network, machine-config, and monitoring operators.

### Q74

**Question:** How do degraded authentication operators affect cluster access patterns?

**Answer:** Auth operator degradation causes login/token issuance issues and can break OAuth-integrated workflows.

### Q75

**Question:** What are common LDAP/OIDC integration failures and quick rollback options?

**Answer:** Common failures: bad bind creds, CA trust, DN filters, unreachable endpoint; keep local break-glass auth path.

### Q76

**Question:** How do you troubleshoot OAuth login loops and callback URL mismatches?

**Answer:** Validate redirect URIs, route/cert trust, oauth client config, cookie domain, and clock skew.

### Q77

**Question:** What causes image registry push/pull failures in internal registry setups?

**Answer:** Causes: registry route/cert issues, auth failure, storage backend unavailable, or image policy restrictions.

### Q78

**Question:** How do you recover from registry storage backend latency or outage?

**Answer:** Restore storage path, verify registry operator health, recover PVC/backing store, and retry with integrity checks.

### Q79

**Question:** What is the role of image pruner, and how can bad pruning break workloads?

**Answer:** Pruner removes old images; wrong retention can delete still-needed layers and break pulls.

### Q80

**Question:** Why do builds fail intermittently when node disk usage is high?

**Answer:** High disk pressure slows layer unpacking/log writes, causing build timeouts and runtime instability.

### Q81

**Question:** What is the difference between BuildConfig failure and deployment failure?

**Answer:** BuildConfig failure is image creation pipeline issue; deployment failure is runtime/scheduling/probe/config issue.

### Q82

**Question:** How do you debug failing init containers in CI/CD workloads?

**Answer:** Inspect init container logs/events, image pulls, config/secret mounts, and dependency reachability.

### Q83

**Question:** Why does a rollout hang at `Progressing=True` but never complete?

**Answer:** Often readiness never passes, wrong maxUnavailable/surge settings, failing hooks, or blocked PDB.

### Q84

**Question:** How do you detect readiness probe flapping due to downstream dependency latency?

**Answer:** Correlate readiness failures with DB/cache latency and tune probe thresholds/timeout/success criteria.

### Q85

**Question:** What causes random pod OOMKills even with apparently sufficient limits?

**Answer:** Memory spikes, JVM/native leaks, sidecar overhead, or node pressure can OOM despite average headroom.

### Q86

**Question:** How do Linux memory overcommit and cgroup limits interact in containerized apps?

**Answer:** Kernel overcommit allows allocations; cgroup hard limits still enforce kill when container exceeds limit.

### Q87

**Question:** What are top causes of high context switching and CPU throttling in pods?

**Answer:** CPU limits too low cause throttling; noisy neighbors and high context switching reduce effective throughput.

### Q88

**Question:** How can `ulimit` or PID limits trigger hidden app instability?

**Answer:** Low file/PID/process limits can crash apps under burst load; check `/proc` limits and node defaults.

### Q89

**Question:** What are common causes of node filesystem inode exhaustion?

**Answer:** Excessive tiny files/logs exhaust inodes before disk space, breaking writes and container startup.

### Q90

**Question:** How do you diagnose log growth causing node disk pressure?

**Answer:** Use node filesystem metrics + container log paths; rotate/compress/ship logs and enforce retention.

### Q91

**Question:** What is the impact of failed MachineConfig rollout on node pools?

**Answer:** Failed MCO rollout leaves mixed node configs and can break cluster consistency/upgrade flow.

### Q92

**Question:** How do you recover when a node is stuck in `Updating` from MCO?

**Answer:** Check MCD logs, rendered config diff, reboot requirements, and failing OS/state drift conditions.

### Q93

**Question:** How do you triage upgrade stalls at 40-70% completion?

**Answer:** Investigate degraded operators, blocked drains, unavailable pools, and incompatible workload constraints.

### Q94

**Question:** What pre-upgrade validations prevent most OpenShift upgrade failures?

**Answer:** Pre-check operator health, node capacity, PDB posture, cert validity, storage/network health, and backup readiness.

### Q95

**Question:** Why is disconnected cluster upgrade often failing at catalog/operator sync?

**Answer:** Mirror/cat source mismatch, missing images/signatures, trust bundle errors, or proxy/firewall constraints.

### Q96

**Question:** Which ports and proxies must be validated for disconnected environments?

**Answer:** Validate API/registry/mirror egress via proxy, DNS resolution, and required control-plane/data ports.

### Q97

**Question:** How do you troubleshoot CSI driver failures for attach/mount operations?

**Answer:** Check CSI controller/node logs, attachment objects, node plugin registration, and backend credentials/network.

### Q98

**Question:** What are recurring Ceph/NFS storage failure reasons in production clusters?

**Answer:** Ceph/NFS failures include network latency, MON quorum issues, permission mismatch, and backend saturation.

### Q99

**Question:** How do you build an intermediate-level incident timeline for postmortems?

**Answer:** Build timeline from alerts, changes, events, logs, metrics, and remediation actions with exact timestamps.

### Q100

**Question:** What are the top 10 intermediate failure patterns you automate alerts for?

**Answer:** Alert on API latency, etcd health, operator degraded, node pressure, PVC pending, route 5xx, and auth failures.

## Section C — Advanced L3 Deep-Dive (Q101-Q150)

### Q101

**Question:** Design a full L3 triage workflow for API outage with partial worker health.

**Answer:** Start blast-radius triage, restore API path, classify control-plane vs infra issue, stabilize, then recover workloads.

### Q102

**Question:** How do you isolate control plane failure from external load balancer failure?

**Answer:** Test API endpoint locally on masters, compare LB health checks/targets, and verify DNS/LB listener paths.

### Q103

**Question:** What are advanced diagnostics for intermittent API server `EOF` and timeout errors?

**Answer:** Capture API server logs/audit with client source correlation; check TLS resets, LB idle timeouts, and etcd stalls.

### Q104

**Question:** How do you correlate API latency with etcd fsync and network RTT metrics?

**Answer:** Correlate apiserver request metrics with etcd fsync/commit latency and node/network RTT histograms.

### Q105

**Question:** What is your L3 playbook for repeated etcd leader elections under peak load?

**Answer:** Verify clock sync, network jitter, disk latency, quorum stability; remove unstable member only with quorum-safe plan.

### Q106

**Question:** How do you decide between etcd defrag, member replacement, or infra scaling?

**Answer:** Defrag for fragmentation, replace only faulty member, scale infra when sustained resource saturation is proven.

### Q107

**Question:** Which OS kernel parameters commonly impact high-scale OpenShift control planes?

**Answer:** Key params include conntrack table, VM dirty ratios, TCP backlog, ephemeral port ranges, and filesystem tuning.

### Q108

**Question:** How do you tune `sysctl` safely for throughput without destabilizing workloads?

**Answer:** Benchmark in staging, change one knob at a time, track SLO impact, and keep rollback of sysctl profiles.

### Q109

**Question:** What are kernel-level causes of dropped packets affecting pod networking?

**Answer:** Causes: NIC offload bugs, MTU mismatch, conntrack overflow, IRQ imbalance, and kernel regressions.

### Q110

**Question:** How do conntrack exhaustion and NAT table limits manifest in OpenShift traffic?

**Answer:** Conntrack exhaustion causes random drops/timeouts; increase limits and reduce churn with keepalive/tuning.

### Q111

**Question:** Which ports are mandatory for OVN, kubelet, API, ingress, monitoring, and logging paths?

**Answer:** Minimum core paths: API `6443`, etcd `2379/2380`, kubelet `10250`, MCS `22623`, DNS `53`, ingress `80/443`.

### Q112

**Question:** How do you perform end-to-end port path validation for north-south traffic?

**Answer:** Validate hop-by-hop with synthetic probes from client->LB->router->service->pod and return path checks.

### Q113

**Question:** How do you debug TLS handshake failures between router and backend service pods?

**Answer:** Check cert chain/SAN, backend TLS settings, route termination mode, and router cipher/SNI configuration.

### Q114

**Question:** What causes route admitted status true but persistent 503 responses?

**Answer:** Usually endpoints are not Ready/mis-targeted or backend port mismatch despite route admission success.

### Q115

**Question:** How do you diagnose mTLS and SNI issues in service mesh + OpenShift route chains?

**Answer:** Align mesh mTLS policy, route passthrough/reencrypt mode, destination rules, and SNI hostnames.

### Q116

**Question:** How do you implement blue-green with strict rollback RTO under 2 minutes?

**Answer:** Pre-provision green, warm caches, run health gates, atomic traffic switch, and pre-tested rollback command.

### Q117

**Question:** How do you validate data consistency during blue-green cutover for stateful services?

**Answer:** Use compatibility contracts, shadow traffic/read checks, consistency metrics, and controlled cutover windows.

### Q118

**Question:** What is your strategy for dual-write and schema compatibility in zero-downtime DB migration?

**Answer:** Do additive schema first, deploy compatible app, dual-write/read-fallback, then remove legacy columns later.

### Q119

**Question:** How do you instrument canary analysis with SLO-based promotion gates?

**Answer:** Use error rate/latency/saturation SLOs with automated promotion/abort thresholds.

### Q120

**Question:** Which failure signals should auto-trigger rollback in GitOps pipelines?

**Answer:** Trigger rollback on sustained 5xx spike, latency breach, failed critical checks, or business KPI regression.

### Q121

**Question:** How do you prevent ArgoCD drift in clusters with manual emergency changes?

**Answer:** Enforce Git as source of truth, reconcile frequently, and use documented break-glass with post-incident back-merge.

### Q122

**Question:** What are top ArgoCD sync failure root causes at enterprise scale?

**Answer:** Common roots: bad repo creds, invalid manifests, CRD ordering, webhook blocks, RBAC denies, cluster drift.

### Q123

**Question:** How do you shard ArgoCD controllers for 500+ apps and multi-cluster targets?

**Answer:** Partition apps by project/cluster, scale repo-server/controller, and isolate high-churn app groups.

### Q124

**Question:** How do you secure ArgoCD repo credentials, tokens, and signing verification?

**Answer:** Use short-lived tokens, sealed/external secrets, signed commits/images, and strict RBAC/audit.

### Q125

**Question:** What are the failure modes of ApplicationSet generators in dynamic cluster fleets?

**Answer:** Generator failures from stale cluster secrets, label mismatches, API rate limits, or template errors.

### Q126

**Question:** How do you design RBAC boundaries across platform team, app team, and security team?

**Answer:** Use least privilege by namespace/project and operation (read/sync/admin), with separate platform break-glass roles.

### Q127

**Question:** How do SCC, PSA, and admission webhooks interact in modern OpenShift hardening?

**Answer:** SCC/PSA/admission must be policy-aligned; conflicting controls create false denies or insecure exceptions.

### Q128

**Question:** What causes admission webhook latency to impact cluster-wide deployment throughput?

**Answer:** Slow webhooks increase admission latency globally, delaying pod creation and rollout completion.

### Q129

**Question:** How do you diagnose webhook deadlocks and fail-open vs fail-close risk?

**Answer:** Define timeout/failure policy carefully; fail-close for security-critical, fail-open for availability-critical non-security paths.

### Q130

**Question:** How do you perform forensic analysis after suspicious lateral movement across namespaces?

**Answer:** Isolate workload, collect audit/network/auth logs, inspect tokens/secrets/RBAC and suspicious exec/network activity.

### Q131

**Question:** What logs and audit events are mandatory for compliance-grade incident reconstruction?

**Answer:** Keep API audit, auth, router, node, and change-management logs with immutable retention.

### Q132

**Question:** How do you design immutable evidence collection in OpenShift security incidents?

**Answer:** Export signed/hashed evidence to write-once storage with chain-of-custody metadata.

### Q133

**Question:** What are top causes of control plane CPU saturation and remediation sequence?

**Answer:** Analyze request storms, watch/list abuse, admission latency, etcd slowness, and optimize clients/controllers.

### Q134

**Question:** How do you determine if bottleneck is app, platform, network, or storage under pressure?

**Answer:** Use RED+USE metrics and dependency graph to pinpoint bottleneck domain before changing infra.

### Q135

**Question:** What are advanced causes of kubelet instability on specific kernel/runtime versions?

**Answer:** Kernel/runtime mismatch, cgroup driver incompatibility, seccomp/SELinux regressions, or filesystem bugs.

### Q136

**Question:** How do you recover from widespread node NotReady caused by bad MachineConfig push?

**Answer:** Pause rollout, select known-good rendered config, recover nodes pool-by-pool with strict canary.

### Q137

**Question:** How do you execute safe rollback when all worker pools are partially degraded?

**Answer:** Prioritize control-plane stability, rollback one worker pool at a time, maintain capacity and PDB compliance.

### Q138

**Question:** What is your plan for region-level cloud outage impacting OpenShift IPI components?

**Answer:** Activate DR runbook: shift traffic, recreate critical infra, restore state, and revalidate IAM/DNS/certs.

### Q139

**Question:** How do you architect multi-region failover for low RPO and low RTO?

**Answer:** Use active-passive or active-active with replicated state, DNS/LB failover, and tested recovery automation.

### Q140

**Question:** How do you validate disaster recovery readiness beyond backup success messages?

**Answer:** Perform full restore drills with production-like data and measure real RTO/RPO, not just backup completion.

### Q141

**Question:** What are common reasons backups are unusable during real restore events?

**Answer:** Backups fail restores due to version drift, missing secrets/keys, incomplete snapshots, or untested procedures.

### Q142

**Question:** How do you validate etcd snapshots for actual restorability and consistency?

**Answer:** Periodically restore snapshots in isolated environment and verify cluster object integrity and service functionality.

### Q143

**Question:** What is your L3 approach for intermittent storage latency spikes causing app timeouts?

**Answer:** Correlate storage latency with pod/app timeouts, queue depth, and backend health; apply QoS/isolation fixes.

### Q144

**Question:** How do you distinguish CSI control plane issue vs backend array/Ceph issue quickly?

**Answer:** If CSI control plane unhealthy cluster-wide, suspect plugin/controller; if localized volumes, suspect backend path.

### Q145

**Question:** Which OS-level metrics best predict node failure before Kubernetes surfaces conditions?

**Answer:** Early predictors: filesystem latency, inode trends, kernel errors, packet drops, conntrack near-max, and OOM trends.

### Q146

**Question:** How do you build predictive alerting to catch recurring failure reasons early?

**Answer:** Build predictive alerts from trend slopes and anomaly models, not only static thresholds.

### Q147

**Question:** How do you standardize root-cause taxonomy for OpenShift incidents across teams?

**Answer:** Standardize RCA tags by domain, trigger, detection gap, and corrective class to improve repeat learning.

### Q148

**Question:** How do you convert top recurring incidents into self-healing automation runbooks?

**Answer:** Convert top incidents to runbooks + automation with guarded actions, approvals, and observability feedback loops.

### Q149

**Question:** How do you evaluate platform maturity from beginner operations to L3 excellence?

**Answer:** Measure maturity by SLO attainment, MTTR, change failure rate, automation coverage, and DR/security readiness.

### Q150

**Question:** If given a failing production cluster, what exact first 30 minutes L3 actions do you execute?

**Answer:** First 30 min: stabilize API/auth/network, stop blast radius, gather evidence, execute proven rollback/recovery path.

