# Documentation Index

This page defines the **canonical documentation map** for this repository.
Use it as the source of truth for active pages and to avoid duplicate content drift.

## Core Repository Docs

- `README.md`: Repository overview and navigation.
- `docs-index.md`: Canonical list of active documentation pages.

## Observability Stack

- `observability/alertmanager/alertmanager.yml`: Alert routing configuration.
- `observability/grafana/slo-dashboard.json`: SLO dashboard definition.
- `observability/loki/loki-config.yaml`: Loki server configuration.
- `observability/loki/promtail-config.yaml`: Promtail log shipping configuration.
- `runbooks/loki-logs-missing.md`: Loki troubleshooting runbook.

## OpenShift Installation and Operations

- `openshift-installation-methods/ipi-installation.md`: Installer-provisioned installation flow.
- `openshift-installation-methods/agent-based-installation.md`: Agent-based installation.
- `openshift-installation-methods/single-node-openshift.md`: Single-node OpenShift setup.
- `openshift-installation-methods/microshift-installation.md`: MicroShift installation guidance.
- `openshift-installation-methods/disconnected-installation.md`: Air-gapped installation flow.
- `openshift-on-premises/installation.md`: On-prem installation guide.
- `openshift-on-premises/administration.md`: On-prem administration guide.
- `openshift-on-premises/troubleshooting.md`: On-prem troubleshooting guide.

## Automation and Deployment Strategy

- `openshift-ansible-installation.md`: Ansible-based OpenShift automation.
- `openshift-terraform-installation.md`: Terraform-based OpenShift provisioning.
- `openshift-gitops-argocd.md`: GitOps with ArgoCD.
- `openshift-blue-green-deployment.md`: Blue-green deployment strategies.
- `openshift-cicd-integration.md`: CI/CD integration patterns.

## Interview Preparation (Canonical)

- `openshift-l3-interview-qa-150.md`: Consolidated 150-question category-wise Q&A bank.

## Canonical Content Rules

- Keep one canonical page per topic.
- If content is merged into a canonical page, remove older split pages.
- Update `README.md` and `docs-index.md` together whenever docs are added/removed.
- Prefer extending canonical pages rather than creating overlapping files.
