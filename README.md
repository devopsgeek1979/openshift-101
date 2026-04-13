# Ultimate OpenShift Observability Platform

This project provides a comprehensive observability stack for OpenShift clusters, integrating Prometheus, Grafana, Loki, Alertmanager, and SLO dashboards to monitor, visualize, and alert on application and infrastructure metrics and logs.

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

## OpenShift On-Premises

This repository also includes comprehensive guides for OpenShift Container Platform on-premises deployments:

- [Installation Guide](openshift-on-premises/installation.md): Step-by-step installation procedures
- [Administration Guide](openshift-on-premises/administration.md): Day-to-day cluster management tasks
- [Troubleshooting Guide](openshift-on-premises/troubleshooting.md): Common issues and resolution steps

## Contributing

1. Fork the repo
2. Create a feature branch
3. Submit a PR

## License

MIT
