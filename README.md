# Ultimate OpenShift Observability Platform

This project provides a comprehensive observability stack for OpenShift clusters, integrating Prometheus, Grafana, Loki, Alertmanager, and SLO dashboards to monitor, visualize, and alert on application and infrastructure metrics and logs.

## Documentation Index

- [Canonical Documentation Map](docs-index.md): Source-of-truth list of active pages and documentation ownership to avoid duplicate content drift

## Features

- **Prometheus**: Metrics collection and alerting (via OpenShift Prometheus Operator)
- **Grafana**: Visualization dashboards, including SLO dashboards
- **Loki**: Log aggregation and querying
- **Promtail**: Log shipping to Loki
- **Alertmanager**: Alert routing and notification management
- **Runbooks**: Troubleshooting guides for common issues

## Prerequisites

- OpenShift 4.x cluster with cluster-admin access
- OpenShift CLI (`oc`) installed
- Helm (optional, for advanced deployments)
- Access to OpenShift monitoring stack (Prometheus Operator)

## Installation

1. Clone this repository:

   ```bash
   git clone <repo-url>
   cd faang-openshift-observability-ultimate
   ```

2. Run the setup script:

   ```bash
   ./scripts/start-observability.sh
   ```

   This script will:
   - Deploy Loki and Promtail using OpenShift templates or operators
   - Configure Alertmanager routes
   - Import Grafana dashboards

## Configuration

### Alertmanager

Edit `observability/alertmanager/alertmanager.yml` to configure alert routing.

### Grafana

The SLO dashboard is defined in `observability/grafana/slo-dashboard.json`. Import it into Grafana.

### Loki and Promtail

Configurations are in `observability/loki/`. Adjust ports and storage as needed.

## Usage

- Access Grafana: `oc get routes -n openshift-monitoring` (or your namespace)
- Query logs with Loki
- View alerts in Alertmanager

## Runbooks

See `runbooks/` for troubleshooting guides:

- [Loki Logs Missing](runbooks/loki-logs-missing.md)

## OpenShift Installation Methods

This repository includes comprehensive guides for various OpenShift installation methods:

### Core Installation Methods

- [User-Provisioned Infrastructure (UPI)](openshift-on-premises/installation.md) - Manual infrastructure setup
- [Installer-Provisioned Infrastructure (IPI)](openshift-installation-methods/ipi-installation.md) - Automated cloud infrastructure
- [Agent-Based Installation](openshift-installation-methods/agent-based-installation.md) - ISO-based installation
- [Assisted Installer](openshift-on-premises/installation.md) - Web-based guided installation

### Specialized Deployments

- [Single-Node OpenShift (SNO)](openshift-installation-methods/single-node-openshift.md) - Compact single-node deployment
- [MicroShift](openshift-installation-methods/microshift-installation.md) - Lightweight edge deployment
- [Disconnected Installation](openshift-installation-methods/disconnected-installation.md) - Air-gapped environments

## OpenShift On-Premises

This repository also includes comprehensive guides for OpenShift Container Platform on-premises deployments:

- [Installation Guide](openshift-on-premises/installation.md): Step-by-step installation procedures
- [Administration Guide](openshift-on-premises/administration.md): Day-to-day cluster management tasks
- [Troubleshooting Guide](openshift-on-premises/troubleshooting.md): Common issues and resolution steps

## OpenShift L3 Administrator Interview Preparation

This repository includes comprehensive interview questions and scenarios for L3 OpenShift administrators:

- [L3 Interview Combined Q&A (150)](openshift-l3-interview-qa-150.md): Single category-wise file containing all 150 questions with mapped answers (beginner, intermediate, advanced L3)

## Infrastructure Automation

This repository includes automation guides for deploying OpenShift using Infrastructure as Code tools:

### Ansible Automation

- [OpenShift Installation with Ansible](openshift-ansible-installation.md): Comprehensive guide for automating OpenShift deployment using Ansible playbooks, including disconnected installations and multi-cluster management

### Terraform Infrastructure

- [OpenShift Installation with Terraform](openshift-terraform-installation.md): Infrastructure as Code guide for deploying OpenShift on AWS and Azure using Terraform, including ROSA and ARO managed services

## GitOps and Advanced Deployments

This repository includes comprehensive guides for GitOps practices and advanced deployment strategies:

### GitOps with ArgoCD

- [OpenShift GitOps with ArgoCD](openshift-gitops-argocd.md): Complete guide for implementing GitOps on OpenShift using ArgoCD, including application deployment, security, and CI/CD integration

### Blue-Green Deployments

- [OpenShift Blue-Green Deployment Strategies](openshift-blue-green-deployment.md): Comprehensive guide for implementing blue-green deployments on OpenShift, including manual and automated approaches, service mesh integration, and rollback strategies

### CI/CD Integration

- [OpenShift CI/CD Integration Guide](openshift-cicd-integration.md): Complete guide for integrating CI/CD tools (Tekton, Jenkins, GitHub Actions) with OpenShift, including DevSecOps practices and multi-cluster deployments

## Contributing

1. Fork the repo
2. Create a feature branch
3. Submit a PR

## License

MIT
