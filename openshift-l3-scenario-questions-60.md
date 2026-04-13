# OpenShift Scenario-Based Interview Questions (60)

This bank contains **60 scenario-based questions** from beginner to L3.
Focus areas include OS diagnostics, ports/connectivity, and common failure reasons.

## Section A — Beginner Scenarios (1-20)

- **S1:** A new app deployment is stuck in `ImagePullBackOff` right after release. What is your step-by-step triage?
- **S2:** Users report route URL works intermittently with occasional `503`. How do you isolate router vs backend issue?
- **S3:** A node flips to `NotReady` after a reboot. What checks do you run in the first 10 minutes?
- **S4:** Pods cannot resolve service DNS names in one namespace only. How do you debug?
- **S5:** Team accidentally removed a secret used by production pods. How do you recover safely?
- **S6:** After adding resource limits, an API service starts timing out. What do you verify first?
- **S7:** A PVC remains `Pending` in dev while same manifest works in staging. What differences do you compare?
- **S8:** SSH to workers works, but app traffic still fails externally. Which ports and layers do you validate?
- **S9:** A workload crashes every restart and enters `CrashLoopBackOff`. How do you decide app bug vs platform issue?
- **S10:** New developers cannot deploy due to SCC denial. How do you resolve least-privilege access?
- **S11:** A deployment rollout is stuck in `Progressing`. What evidence do you gather before rollback?
- **S12:** External DNS points correctly, but API endpoint is unreachable. What path checks do you perform?
- **S13:** Router pod is healthy, but one route is never admitted. How do you investigate?
- **S14:** After changing probe settings, traffic dropped 40%. How do you revert and validate fix?
- **S15:** A namespace hits quota unexpectedly. How do you identify top consumers quickly?
- **S16:** CI pods fail with permission denied writing cache files. Which security and filesystem checks matter?
- **S17:** A service has endpoints but clients still timeout. What network diagnostics do you run?
- **S18:** App can call DB from one node but not another. What node-level and policy-level checks do you perform?
- **S19:** Internal registry pulls fail for one project only. How do you isolate RBAC vs auth vs network?
- **S20:** Team asks for root containers to “fix quickly.” How do you handle this securely?

## Section B — Intermediate Scenarios (21-40)

- **S21:** Cluster autoscaler does not add nodes despite pending pods. What are your top root-cause checks?
- **S22:** Upgrade from 4.x to next minor stalls at 55%. How do you triage operator dependencies?
- **S23:** After MachineConfig update, 20% workers are stuck in Updating. What recovery plan do you execute?
- **S24:** A strict NetworkPolicy rollout breaks metrics scraping. How do you restore safely without opening everything?
- **S25:** LDAP auth intermittently fails while local `kubeadmin` works. What signals confirm IdP-side issue?
- **S26:** A stateful app reports I/O timeout spikes every evening. How do you prove storage backend bottleneck?
- **S27:** Team reports random 502/504 through ingress during scale test. How do you correlate router and app metrics?
- **S28:** Build pipelines fail during image push after registry maintenance. What checks are mandatory?
- **S29:** Frequent OOMKills started after enabling sidecars. How do you rebalance requests/limits?
- **S30:** etcd warning alarms appear but user traffic looks normal. Do you act now or wait, and why?
- **S31:** Disconnected cluster cannot install operators after mirror update. What mirror/catalog validation do you run?
- **S32:** Pods in one AZ fail to mount volumes, others are fine. How do you isolate infra vs CSI issue?
- **S33:** A team accidentally applied deny-all egress to production namespace. What is your immediate response?
- **S34:** Node drains fail due to PDB conflicts during patching window. How do you complete maintenance safely?
- **S35:** API latency rises after enabling new admission webhook. How do you confirm webhook impact?
- **S36:** Route works from internet but fails from corporate network. Which firewall/proxy checks come first?
- **S37:** Container startup is slow only on newly provisioned workers. Which OS/runtime checks are likely causes?
- **S38:** Blue deployment is stable, green passes smoke tests, but post-switch errors increase. What hidden dependencies do you verify?
- **S39:** One team bypassed GitOps and changed live manifests manually. How do you reconcile drift without outage?
- **S40:** Repeated pod rescheduling causes user session drops. What app and platform controls do you improve?

## Section C — Advanced L3 Scenarios (41-60)

- **S41:** API is intermittently unreachable, etcd reports leader changes, and control-plane CPU is high. Walk through your L3 war-room sequence.
- **S42:** You suspect conntrack exhaustion causing random pod connection resets. How do you prove and mitigate fast?
- **S43:** Multi-cluster GitOps sync flood causes ArgoCD controller lag. What scaling and sharding strategy do you apply?
- **S44:** Critical cert expires in 18 hours with strict change control. How do you rotate with minimal risk?
- **S45:** During region outage, half the services fail over incorrectly due to stale DNS. What failover controls do you redesign?
- **S46:** A bad MachineConfig made many nodes NotReady. How do you rollback while preserving cluster capacity?
- **S47:** Security incident shows suspicious cross-namespace access from one service account. What forensic and containment steps do you execute?
- **S48:** etcd disk fsync latency spikes under burst load. What short-term and long-term remediations do you choose?
- **S49:** Canary release passes app metrics but business KPIs drop. How do you improve promotion gates?
- **S50:** mTLS-enabled service mesh and route passthrough conflict after certificate renewal. How do you debug chain-of-trust?
- **S51:** Persistent 503 on one route despite healthy pods and service endpoints. What deeper checks reveal root cause?
- **S52:** DR drill backup restores succeed technically but app data is inconsistent. How do you redesign backup validation?
- **S53:** Cluster upgrade repeatedly fails only in one environment with same manifests. How do you identify environmental drift?
- **S54:** High-priority workload starves due to noisy neighbors despite quotas. What scheduling and node-pool strategy do you propose?
- **S55:** You must patch a critical CVE across 12 clusters in 24 hours. How do you orchestrate staged updates and rollback?
- **S56:** Large-scale rollout causes OAuth login latency and token failures. Which components do you analyze first?
- **S57:** Multiple teams need privileged access for troubleshooting. How do you provide secure, auditable break-glass workflow?
- **S58:** Platform SLO breaches occur weekly with no single obvious cause. How do you build a recurring-failure taxonomy?
- **S59:** A stateful blue-green cutover must finish in under 3 minutes with zero data loss. What architecture and runbook do you use?
- **S60:** You inherit an unstable production cluster with incomplete docs. What are your first 30-minute, 24-hour, and 7-day actions?

## Optional Interviewer Prompts

- Ask the candidate for **first 5 commands**, **top 3 hypotheses**, and **rollback plan**.
- Ask which **ports** and **dependencies** must be validated before concluding RCA.
- Ask what **prevention automation** they would implement after fixing the issue.
